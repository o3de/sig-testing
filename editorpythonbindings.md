---
description: ' Follow these steps to use or extend EditorPythonBindings. '
title: Using & Extending EditorPythonBindings
---

EditorPythonBindings (EPB) offers access to build scalable multi-point, scriptable, end-to-end, round-trip pipelines, and common framework solutions for game content creators and Open 3D Engine (O3DE) contributors via Python. These solutions will measurably drive things like:

* Improve extensibility by reducing the barrier of entry for building out new custom workflows or pipelined processes
* Increase efficiency via automation to reduce the number of clicks to operate
* Allow automated content to process and run hands-free

## Using EPB with Tests
While the O3DE public documentation should serve as the primary source for usage, here is a brief example of how to use EPB in a test. You'll need the QtForPython gem in order to continue.

This code snippet is the entry point into the entire object/widget hierarchy of our Qt application.  From here, you can do things like access the menu bars/toolbars, find children widgets to inspect/inject events onto, and more.

```python 
# Returns a PySide2 wrapped instance of our QMainWindow
editor_window = pyside_utils.get_editor_main_window()
```

As you can see, you can treat EPB as you can most other Python modules.

## Extending EPB

### Overview
The EPB gem is an optional gem. This means that when a project wants to expose scriptable functionality to Python, a project team needs to enable the “Editor Python Bindings” gem for the project. The next step is for the feature team to add behavior scoped for automation.

The gem does not automatically expose behavior classes, buses, or properties to Python. Instead it offers mechanisms to expose these behaviors to Python using a combination of attributes to flag behavior to expose to Python. This means that a C++ team has to update their code to use these new attributes so that scripters can use the behavior in Python scripts to automate content creation and/or tests.

### The Attributes
The Script Context has three new attributes to expose behavior for automation usage. The gem uses the Scope attribute to set behavior to usage in the Launcher, the Editor, or for Common.

**Scope and ScopeFlags**
```c++
AZ::Script::Attributes::Scope
AZ::Script::Attributes::ScopeFlags::Launcher
AZ::Script::Attributes::ScopeFlags::Automation
AZ::Script::Attributes::ScopeFlags::Common
```

The feature teams exposes either a Behavior Class or Behavior EBus by using the Script Scope attribute with either the Automation or Common flag.

For example:
```c++
if (AZ::BehaviorContext* behaviorContext = azrtti_cast<AZ::BehaviorContext*>(context))
{
    behaviorContext->Class<MyType>("MyType")
        ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
        ->Attribute(AZ::Script::Attributes::Module, "types")
        ;
}
```

This exposes the MyType class to the behavior context (as a Behavior Class) with two attributes:

    1. Scope with the Automation flag
    2. Module with the text "types"


This will expose the MyType class to the azlmbr.types module. It is possible to scope Behavior EBuses, Classes, constants, enums, and properties for Automation.

A full example for a reflect function might look like this:

```c++	
void Reflect(AZ::ReflectContext* context)
{
    if (AZ::SerializeContext* serializeContext = azrtti_cast<AZ::SerializeContext*>(context))
    {
        serializeContext->Class<MyType>()
            ->Version(1)
            ->Field("Data", &MyType::m_data)
    }
     
    if (AZ::BehaviorContext* behaviorContext = azrtti_cast<AZ::BehaviorContext*>(context))
    {
        behaviorContext->Class<MyType>("MyType")
            ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
            ->Attribute(AZ::Script::Attributes::Module, "types")
            ->Method("DoWork", &MyType::DoWork)
              ->Attribute(AZ::Script::Attributes::Alias, "do_work")
            ->Property("Data", &MyType::GetData, &MyType::SetData)
              ->Attribute(AZ::Script::Attributes::Alias, "data")
            ;
    }
}
```

##### Scoping For In Game Mode
There are actually two separate game client modes 1) the game Launcher run time and 2) the game mode inside the Editor. The Python scripts will not work in the first client mode since the Python Virtual Machine (VM) is not loaded in the Launcher.

The second mode can run Python script code but it is not fully supported; so you can use Python code in the Editor's game mode but not all behavior has been set up to work in this game mode.

The default Scope value is AZ::Script::Attributes::ScopeFlags::Launcher since the scripting contexts for Lua and Scripting Canvas where the first mechanisms for binding behavior. The AZ::Script::Attributes::ScopeFlags::Common and AZ::Script::Attributes::ScopeFlags::Automation flags also have important contexts and mechanisms.

### Module
```c++
AZ::Script::Attributes::Module
```
The Editor Python Bindings gem uses the azlmbr base package for Python bindings for scripts to import.

The Module attribute exposes the package that the behavior to Python. The attribute accepts a text string to be used add the types, methods, and properties to a Python package. The text string can contain dotted notation to make sub-packages such as 'legacy.general' exports the behavior to azlmbr.legacy.general. The gem uses this attribute to organize types, buses, and static methods into a specific packages.

```c++
behaviorContext->Class<MyType>("MyType")
  ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
  ->Attribute(AZ::Script::Attributes::Module, "types");
```

This code puts the "MyType" class type into the azlmbr.types module. A scripter can dir() the module to discover the class types that are inside the module.

If the coder specifies no module then the gem will place the static methods into the azlmbr.default package.

### Alias
```c++
AZ::Script::Attributes::Alias
```
You can give methods and properties more Python-ic names to behavior classes using the Alias attribute. The coding standard for naming methods in the Behavior Context is to match the name of the C++ native method name, but the standard in Python is PEP8 snake case. To assign a script friendly name the code looks like this:

```c++
behaviorContext->Class<MyType>("MyType")
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
    ->Attribute(AZ::Script::Attributes::Module, "types")
    ->Method("DoWork", &MyType::DoWork)
    ->Attribute(AZ::Script::Attributes::Alias, "do_work")
    ->Property("theData", &MyType::GetData, &MyType::SetData)
    ->Attribute(AZ::Script::Attributes::Alias, "data");
```

The operation effectively renames the type's method and property name to match PEP 8 standards. The above code makes this Python code possible:

```python
import azlmbr.types
instance = azlmbr.types.MyType()
instance.data = 'text data'
instance.do_work([1, 2, 3])
```

### Behavior Class as Python Types and Methods
The gem enumerates all the Behavior Class types and exposes static methods in the specified module and class types with properties and/or member functions as proper Python object types in a specified module.

#### Static Methods
The feature teams can expose static methods to Python but wrapping them into a Behavior Class and declaring Method on a static function.

```c++
behaviorContext->Class<PythonReflectionTestDoPrint>()
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
    ->Attribute(AZ::Script::Attributes::Module, "legacy.work")
    ->Method("do_work", DoWork, nullptr, "Does work in the legacy style.");
```

This code will put the non-member methods (static methods) into the azlmbr.legacy.work package so that the Python code might look like:

```python
import azlmbr.legacy.work as work
work.do_work('some data text')
```

#### Behavior Class Types
The script can create Python object instances either by the azlmbr.object.create() method or by the using the module type name directly where the Module attribute is specified.

```c++
behaviorContext->Class<MyType>("MyType")
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
    ->Attribute(AZ::Script::Attributes::Module, "types")
    ->Method("DoWork", &MyType::DoWork)
    ->Property("theData", &MyType::GetData, &MyType::SetData);
```

Exposing the Behavior Class like that code allows for this type of Python:

```python
import azlmbr.types
instance = azlmbr.types.MyType()
instance.theData = 'text data'
instance.DoWork([1, 2, 3])
```

#### Global Constants and Properties
Feature teams can expose C++ constants, enums, and properties to the Behavior Context.

```c++
behaviorContext
    ->Property("coolProperty", MyGlobals::GetValue, MyGlobals::SetValue)
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation);
 
behaviorContext
    ->EnumProperty<GlobalEnums::GE_LUMBER>("GE_LUMBER")
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation);
 
behaviorContext
    ->ConstantProperty("ONE", []() { return s_one; })
    ->Attribute(AZ::Script::Attributes::Module, "my_constants");
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation);
```

This code will expose these constant properties to Python in these forms:

    * A Constant without a Module attribute: azlmbr.globals.property.coolProperty
    * Constant Enum property with out Module attibute: azlmbr.globals.GE_LUMBER
    * A Constant with a Module attribute: azlmbr.globals.my_constants.ONE



Constant/Enum Name | Module Attribute Name | Final Name | Final Module
:--| :--| :-----| :-----
coolProperty | None | azlmbr.globals.property.coolProperty | azlmbr.globals.property
GE_LUMBER | None | azlmbr.globals.GE_LUMBER | azlmbr.globals 
ONE | my_constants | azlmbr.globals.my_constants.ONE | azlmbr.globals.my_constants

### Behavior EBus to Python Bus
The feature teams can expose access to any EBus using the Behavior EBus declarations. The gem supports request bus calls including broadcast, event, and queue type calls. The gem also supports bus notifications as long as the bus declares a handler type.

#### Writing an EBus or Methods Meant For Scripting
There are some basic rules that C++ coders should used when designing a Behavior EBus interfaces or Class types for Python (or any scripting) usage:
**Do not return naked pointers**

It is not safe to send naked pointers back to script callers since they have no idea when the object lifetime expires for a native instance.

```c++
struct MyBlock
{
    // some data
};
using MyBlockId = AZ::u32;
 
// Bad
struct MyBlockManagerNaked
    : AZ::EBusTraits
{
    virtual MyBlock* CreateBlock() = 0; // this is bad
    virtual AZStd::string ReadBlockName(MyBlock*) = 0; // the pointer could be invalid
};
 
// Good
struct MyBlockManagerById
    : AZ::EBusTraits
{
    virtual MyBlockId ConstructBlock() = 0; // this can be cached by script
    virtual AZStd::string GetBlockName(MyBlockId blockId) = 0;
};
```

**Do no use in/out parameters in the API**

There is no good way to know that an input parameter is designed to be used both as input and returned as an output when binding C++ native methods to script methods.

```c++
struct MyInputOutputAPI
    : AZ::EBusTraits
{
    virtual bool CreateBlock() = 0; // this is bad
    virtual AZStd::string ReadBlockName(MyBlock*) = 0; // the pointer could be invalid
};
```

#### Bus Type Enumerations
The gem has four built-in values to indicate how to call a bus method.

**azlmbr.bus.Event** - indicates that the bus needs an address to send the data
**azlmbr.bus.QueueEvent** - indicates that the bus needs an address to send the data in the future
**azlmbr.bus.Broadcast** - indicates that the bus sends the data to all connected handlers
**azlmbr.bus.QueueBroadcast** - indicates that the bus sends the data to all connected handlers in the future

#### Request Bus
A team exports a request bus by declaring a bus to the Behavior Context.

```c++
behaviorContext->EBus<MyRequestBus>("MyRequestBus")
    ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
    ->Attribute(AZ::Script::Attributes::Module, "foobar")
    ->Event("DoEventWork", &MyRequestBus::Events::DoEventWork);
```

This basically exports the bus to azlmbr.foobar.MyRequestBus so that Python can call either Broadcast or Event calls to the bus.

The pattern for calling bus calls in Python is:

```python
<return_type> azlmbr.<submodule>.<bus_name>(<call_type>, '<EventName>', [address], data...)
 
if no Module attribute:
 
<return_type> azlmbr.bus.<bus_name>(<call_type>, '<EventName>', [address], data...)
```

If the MyRequestBus is addressable via an integer, then the call might look like:

```python
import azlmbr.bus
import azlmbr.foobar
address = 101
answer = azlmbr.foobar.MyRequestBus(azlmbr.bus.Event, 'DoEventWork', address, 40, 2)
```

If the MyRequestBus is singleton broadcast type of bus, then the call might look like:

```python
import azlmbr.bus
import azlmbr.foobar
answer = azlmbr.foobar.MyRequestBus(azlmbr.bus.Broadcast, 'DoEventWork', 40, 2)
```

**Async bus calls**
The gem offers the ability to call the async versions of the bus methods. If the MyRequestBus is addressable via an integer, then the async call might look like:

```python
import azlmbr.bus
import azlmbr.foobar
address = 101
answer = azlmbr.foobar.MyRequestBus(azlmbr.bus.QueueEvent, 'DoEventWork', address, 40, 2)
```

If the MyRequestBus is singleton broadcast type of bus, then the async call might look like:

```python
import azlmbr.bus
import azlmbr.foobar
answer = azlmbr.foobar.MyRequestBus(azlmbr.bus.QueueBroadcast, 'DoEventWork', 40, 2)
```

The C++ code is still required to call the ExecuteQueuedEvents() method for the bus.

**Notification Bus**
The gem offers the ability to hook into bus notifications when the coder supplies a Handler for an EBus.

This notification bus might look like this:

```c++
struct MyNotifications
    : public AZ::EBusTraits
{
    static const AZ::EBusAddressPolicy AddressPolicy = AZ::EBusAddressPolicy::ById;
    static const AZ::EBusHandlerPolicy HandlerPolicy = AZ::EBusHandlerPolicy::Single;
    using BusIdType = AZ::s32;
    virtual void OnResult(AZ::s64 result) = 0;
};
using MyNotificationBus = AZ::EBus<MyNotifications>;
```

The gem creates an instance of the handler type to bind to the calls which will send the data back to a possible Python handler:

```c++
struct MyNotificationsHandler final
    : public MyNotificationBus::Handler
    , public AZ::BehaviorEBusHandler
{
    AZ_EBUS_BEHAVIOR_BINDER(MyNotificationsHandler, "{5F091D4B-86C4-4D25-B982-2ECAFD8AFF0F}", AZ::SystemAllocator, OnResult);
 
    void OnResult(AZ::s64 result) override
    {
        Call(FN_OnResult, result);
    }
 
    void Reflect(AZ::ReflectContext* context)
    {
        if (AZ::BehaviorContext* behaviorContext = azrtti_cast<AZ::BehaviorContext*>(context))
        {
            behaviorContext->EBus<MyNotificationBus>("MyNotificationBus")
                ->Attribute(AZ::Script::Attributes::Scope, AZ::Script::Attributes::ScopeFlags::Automation)
                ->Attribute(AZ::Script::Attributes::Module, "notes")
                ->Handler<MyNotificationsHandler>()
                ->Event("OnResult", &MyNotificationsHandler::Events::OnResult)
                ;
        }
    }
};
```

This code allows a script to hook into the 'OnResult' event name, addressable via a s32 number.

```python
import azlmbr.bus
 
def on_result(parameters):
    # the parameters come in as a tuple
    print('result is {}'.format(int(parameters[0])))
 
handler = azlmbr.notes.MyNotificationBusHandler()
handler.connect(101)
handler.add_callback('OnResult', on_result)
 
def disconnect_handler():
    global handler
    handler.disconnect()
```

The handler will get handle 'OnResult' events that are addressed to 101 to the Python function name on_result(). The on_result Python function needs to parse the input of 'parameters' since it is a tuple of data.

### Supported Marshaling Types
The Editor Python Bindings gem supports a wide range of data types that can be marshaled between Python and the Editor.

#### Numeric Types
The gem exports primitive numeric types from C++ including integer, real numbers, and bool; as well as the VectorFloat type. It does not work for atomics or CPU intrinsics (e.g. __m64, __m128i).

#### String Types
The gem also handles string types for AZStd::string, AZStd::string_view, and const char* (to support legacy API functions).

#### Behavior Class Types
The gem exposes Behavior Class types that have been scoped for either Automation or Common usage. The methods and properties are exposed to Python using either the original registered name; or the Python name can be renamed using the Alias attribute.

#### Containers
The plan is to support Python container types like Set, but this is currently not supported. List and Dictionary are supported.

**List**
The gem offers support for Python List containers via exporting AZStd::vector<> behavior arguments or return values. The supported types for lists are numeric types (e.g. int, float, u64), VectorFloat, and flagged behavior class types.

**Dictionary**
The gem offers support for Python Dictionaries via exporting AZStd::unordered_map<> behavior arguments or return values. This supports the same types as the List container types.

**Container Limits**
The Python List is mapped to a C++ vector<> type, so all the underlying contained types must be the same data type; unlike Python where the data types can be changed out.

### Supported Data Types for Marshaling
These are the data types that can be passed in to and out of a Behavior Method or EBus.

Behavior Data Type | Python Type | Notes
:--| :--| :-----
AZStd::any | native type or Proxy Object | The gem attempt to convert to Python native types for ease use
AZStd::string | String |
AZStd::string_view | String | 
const char* | String | Mostly to support legacy Editor functions
bool | Bool | 
signed integer | Integer | int8, int16, int32, int64
unsigned integer | Integer | uint8, uint16, uint32, uint64
float | Float | double precision in Python
double | Float | double precision in Python
AZ::VectorFloat | Float | Exists as a Python Float in the scripts
AZStd::unordered_map<> | Dictionary | might be need to use uniform key-value pairs
AZStd::vector<> | List | must be a uniform supported data type
Behavior Class | Proxy Object | Must be scoped for either Automation or Common
char | Char | Unicode Char is supported

The Editor (or gems) will scope as Automation (or Common ) Behavior Class types like Uuid, AssetId, and AssetId so that scripts will be able to send and get these data types.

### Using the Editor Python Bindings gem in Non-Editor Applications
The EBP gem can also automate any behavior of any application.

The requirements:

    * The tool/application should derive from the AzToolsFramework::ToolsApplication class for tools with UI
    * Be able to load gems
    * Have both a AZ::SerializeContext and AZ::BehaviorContext
    * Have access to the AzToolsFramework::EditorPythonEventsInterface

 After the application launches, has reflected the serialization & behavior contexts, and is ready to execute Python scripts then the application should call:

```c++
auto editorPythonEventsInterface = AZ::Interface<AzToolsFramework::EditorPythonEventsInterface>::Get();
if (editorPythonEventsInterface)
{
    editorPythonEventsInterface->StartPython();
}
```

This starts up the embedded Python virtual machine, runs the bootstrap scripts (if any), and examines the Behavior Context registry to bind the scoped behavior for Python usage.

At this point Python scripts can start making calls to:

```c++
AzToolsFramework::EditorPythonRunnerRequestBus::Broadcast(&AzToolsFramework::EditorPythonRunnerRequestBus::Events::ExecuteByFilenameWithArgs, filename, pythonArgs);
```

When the application is ready to be shutdown the StopPython() can be called, but the system component inside the gem will also automatically shutdown the Python virtual machine on exit; so this step is optional.

```c++
EditorPythonEventsInterface
namespace AzToolsFramework
{
    //! Interface to signal the phases for the Python virtual machine
    class EditorPythonEventsInterface
    {
    public:
        AZ_RTTI(EditorPythonEventsInterface, "{F50AE641-2C80-4E07-B4B3-7CB34FFAB393}");
 
        //! Signal the Python handler to start
        virtual bool StartPython();
 
        //! Signal the Python handler to stop
        virtual bool StopPython();
    };
}
```

### Setup Python Logging
The gem redirects the Python logs to an EBus named AzToolsFramework::EditorPythonConsoleNotificationBus so that the output can be directs to files, tests, or consoles.

EditorPythonConsoleNotificationsBus
```c++
namespace AzToolsFramework
{
    //! A bus to handle post notifications to the console views of Python output
    class EditorPythonConsoleNotifications : public AZ::EBusTraits
    {
    public:
        //////////////////////////////////////////////////////////////////////////
        // EBusTraits overrides
        static const AZ::EBusHandlerPolicy HandlerPolicy = AZ::EBusHandlerPolicy::Multiple;
        static const AZ::EBusAddressPolicy AddressPolicy = AZ::EBusAddressPolicy::Single;
        //////////////////////////////////////////////////////////////////////////
 
        //! post a normal message to the console
        virtual void OnTraceMessage(AZStd::string_view message) = 0;
 
        //! post an error message to the console
        virtual void OnErrorMessage(AZStd::string_view message) = 0;
    };
    using EditorPythonConsoleNotificationBus = AZ::EBus<EditorPythonConsoleNotifications>;
}
```

Example usage code:

```c++
void Script::Init()
{
    AzToolsFramework::EditorPythonConsoleNotificationBus::Handler::BusConnect();
}
 
void Script::OnTraceMessage(AZStd::string_view message)
{
    // do work
}
 
void Script::OnErrorMessage(AZStd::string_view message)
{
    // do work
}
```

### Executing Python Scripts
The AzToolsFramework::EditorPythonRunnerRequestBus executes Python code either by string or by file name. The bus can send an optional number of arguments when it executes with a Python file.

EditorPythonRunnerRequestBus
```c++
namespace AzToolsFramework
{
    /**
      * Provides a bus to run Python scripts
      */
    class EditorPythonRunnerRequests
        : public AZ::EBusTraits
    {
    public:
        //////////////////////////////////////////////////////////////////////////
        // EBusTraits overrides
        static const AZ::EBusHandlerPolicy HandlerPolicy = AZ::EBusHandlerPolicy::Single;
        static const AZ::EBusAddressPolicy AddressPolicy = AZ::EBusAddressPolicy::Single;
        //////////////////////////////////////////////////////////////////////////
  
        //! executes a Python script using a string
        virtual void ExecuteByString(AZStd::string_view script);
  
        //! executes a Python script using a filename
        virtual void ExecuteByFilename(AZStd::string_view filename);
  
        //! executes a Python script using a filename and args
        virtual void ExecuteByFilenameWithArgs(
            AZStd::string_view filename,
            const AZStd::vector<AZStd::string_view>& args);
    };
    using EditorPythonRunnerRequestBus = AZ::EBus<EditorPythonRunnerRequests>;
  
} // namespace AzToolsFramework
```

## Related  Documentation
* [O3DE EPB Documentation](https://o3de.org/docs/user-guide/gems/reference/script/python/editor-python-bindings/)