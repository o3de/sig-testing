# RFC: In-Engine Test Harness Gem

## Summary

O3DE needs test automation tools for the game-executables created via O3DE. An in-engine testing system would simplify verifying features of O3DE, as well as enable all developers using O3DE to automate tests for their own code. In turn, this can improve the experience of every end-user who consumes a product made with O3DE. This is a broad project proposal, and individual parts should be clarified in future RFCs.

## What is the relevance of this feature?

Test automation is key to efficient software development, and is increasingly highlighted as key to success in modern game development workflows ([1](https://www.youtube.com/watch?v=X673tOi8pU8),[2](https://www.youtube.com/watch?v=s1JOSbUR6KE),[3](https://www.youtube.com/watch?v=8d0wzyiikXM),[4](https://www.youtube.com/watch?v=2VDlX3Dqm0w),[5](https://www.gamedeveloper.com/programming/testing-for-game-development#automation)). In the realm of simulations, automating a test can help support or disprove a hypothesis in an easily repeatable way. Tests can also serve as a learning tool, helping document and drive a real-world example of how a developer can use the system under test. Currently there is no simple and automated way to run tests in the game "launcher" executables created via O3DE. Developers are left to implement their own solution. This proposal intends to fill that in-game testing gap.

O3DE's current test-writing tools primarily target C++ libraries at a low level with GoogleTest, or target the O3DE Editor at a higher level with Python-based tools. While developers can always write arbitrary code to run in O3DE launchers, there are no standardized tools to support writing test code in the engine. Providing these tools as a Gem would allow developers to easily onboard to in-game testing, and make it easy to disable that automation before public release.

## Feature design description

Game logic test-writing should meet the developer where they do their work: at the inter-system "gameplay programming" level. Developers add new production logic between O3DE systems in multiple locations:

- Lua Scripts
- ScriptCanvas Scripts
- C++ Components
  - C++ SystemComponents (which could include extending to additional scripting languages)

O3DE needs an easy way to author tests for production code at each location, ideally allowing for tests written in the same language(s) as the production code. These tests must be easy to discard before shipping the final product to customers. It should also be easy for users to filter and run tests, though ease of execution is relative to which development tool the user prefers. Since different developers favor different workflows, each of the following interfaces can be made available to execute and report on tests:

1. Command-line arguments to a CLI test runner tool, which starts the tests in a game executable
   - CMake/CTest, via registration of command-line arguments
   - IDEs, via CMake/CTest targets
   - Continuous Integration Pipelines, via CTest *or* the CLI test runner
2. O3DE Editor, with a UI-based test runner which executes tests by entering game-mode

After running a set of tests, the developer should be presented with a concise report summary. This report should clarify where in code any failures occurred, and the location of relevant logs or stacktraces.

There are many types of in-game tests which would become easier to author. The next few headings clarify some prominent categories:

### Scenario 1: Tests of O3DE features

An example of an in-game test would be a simple [black-box test](https://www.softwaretestinghelp.com/black-box-testing/) for an existing O3DE system, such as Physics:

![Physics test](phystest.png?raw=true)

1. Initialize a scenario
   - load a level
   - spawn two entities with physics components
   - additional configuration (such as making sure they are not asleep/inactive)
2. Run the test
   - apply a force to one entity
   - expect a collision event emitted within one second
3. Clean up the scenario
   - despawn the entities

This test could be summarized as "When objects collide, objects are notified of collision." Such a test could also be extended into more specific tests of how collision is resolved, and split into multiple tests with variant object parameters.

Note: The test above can already be written in the O3DE Editor, through the EditorPythonBindings (EPB) interface. However EPB tests can only run in the context of the Editor, and only on the host system where it is developed. A test using the proposed interface could run in executable that gets deployed to any target system, including mobile devices and consoles.

### Scenario 2: Tests of single-player game logic

Customers of O3DE can also create tests for their own gameplay features, verifying behavior beyond what came bundled with O3DE. For this example, assume someone has created a real-time game where Artificial Intelligence (AI) controlled non-player characters (NPCs) can attack one another. A simple test for this system would be:

![AI test](aitest.png?raw=true)

1. Initialize a scenario
   - load a level
   - spawn two entities with the necessary AI and NPC components
   - temporarily disable the AI on both entities
   - configure them to be ready for combat
     - equip them with any prerequisites such as a weapon
     - enable flags to set them as hostile to one another
2. Run the test
   - activate their AI behaviors
   - expect a damage message to be emitted within 100 frames.
3. Clean up the scenario
   - despawn the entities
   - reset any secondary data tracked by AI or Progression systems

This test could be summarized as "When hostile AI meet, they attack one another and cause damage." Depending on the game design, such a test could also be extended into a deeper test of verifying one of the characters eventually collapses. (this would make the test slower and also more fragile to gameplay changes, if in potentially useful ways which can expose game design issues)

### Scenario 3: Tests of networked game logic

Many games depend on systems networked across multiple machines. And while delivering a securely networked test framework can be excluded from this feature's scope, the proposed system must be designed to exist as part of a distributed framework. Tests of this style would include a higher-level orchestration than the examples above:

![Networked test](nettest.png?raw=true)

1. Configure additional external (cloud) services
2. Set up Server(s)
   1. Deploy executables to server machine(s)
   2. Configure and start server executables to create a session
3. Set up Client(s)
   1. Deploy executables to networked client machine(s)
   2. Configure and start client game executables
   3. Connect networked client(s) to server session(s)
4. Start test scenario
   1. Execute test framework in game or server executable(s) <-- proposed feature, highlighted in blue
      1. Initialize a scenario
      2. Run the test (which can interact with other clients and services)
      3. Clean up scenario locally
   2. Execute post-test verifications (e.g. verify content of server logs, test logs across multiple clients, and/or state of cloud services)
   3. Execute post-test cleanup (e.g. revert secondary changes to client, server, and service state)
   4. Report scenario status
5. Teardown Client(s)
6. Teardown Server(s)
7. Teardown Services

The proposed feature covers only one part of step 4, but needs to be capable of running within the distributed context. There are many subtle parts of authentication and deployment which should be handled by other automated systems. However even without those systems defined, delivering the proposed feature should enable local-only interaction between client and server executables.

## Technical design description

In-engine test tools should create a lightweight interface to the existing ways which developers write gameplay-level code. Different systems inside O3DE are knit together primarily with AZ::Interface, AZ::Event, and/or EBus messages. This is true both for preexisting systems provided with O3DE, as well as new user-defined code. Individual tests will rely on existing interfaces to automate interactions, in the same way as production game code.

The overall test execution and reporting can be coordinated by a SystemComponent specific to the test-system, with individual tests existing as Components attached to Entities in the game world. UI to simplify representation in the Editor requires an EditorSystemComponent. Everything except user-defined test code should exist in an easily disableable Gem, and disabling the Gem must also cleanly disable all user-defined test code that depends on it.

![Overview](overview.png?raw=true)

The diagram above outlines the data flow for in-engine automation, composed of the following separate layers:

1. Remote Tool (standalone)
   - Remote command interface
     - Queues running arbitrary commands in game, such as starting the Test Harness
2. Test Harness Core
   - SystemComponent
     - Define external interface for test-execution requests
     - Define AZ::Event interface for tests to report into
     - Track available tests
   - Generic C++ test-component interface
     - Registers user's test by sending an AZ::Event (to the SystemComponent)
     - Records status from test instances by sending AZ::Event(s)
3. Automated Execution
   - CLI test runner
     - Automates sending multiple commands across the remote tool to the SystemComponent
     - Automates collecting test reports into standard JUnit/xUnit XML format
   - Report generator
     - Records test status from inside game executable to a local report file
4. Basic Test Harness UX
   - CTest registration helper functions
     - Simplifies adding tests to IDE's and CI pipelines such as Jenkins
   - Basic Editor UI for test component placement
     - Simplifies configuring tests attached to specific entities in the game world
   - Gem-component disable pattern
     - Cleanly disable user-defined components when test system is disabled
     - Avoid warning spam and performance impacts

The layers above would provide a minimum viable product. After doing so, the system can extend to additional systems, such as:

5. Lua Scripts
   - Extend C++ test-component adapter to the existing Lua script component
   - Provide boilerplate methods for writing Lua tests
6. ScriptCanvas Scripts
   - Extend C++ test-component adapter to the existing SC component
   - Provide communication nodes for writing SC tests
7. In-Editor testing UX
   - Simplifies test script iteration
     - Display XML reports into the O3DE Editor
     - Simulate tests via Editor game-mode
8. Remote testing features
   - Networked test orchestrator
     - Deploy and coordinate test commands across multiple executables
   - Editor-based UI for networked test orchestrator
     - Build, deploy, and run tests on remote devices such as a mobile phone or build farm
   - Security features
     - Mutual TLS for remote interface tool security
     - Script file hash manifest to prevent file tampering

### Test Orchestration

A SystemComponent can automatically manage test execution and reporting within one executable, though it also needs an interface to the outside world. To easily detect and report crashes, part of the orchestration of tests should exist out-of-process with the O3DE engine. A commandline interface can exist as a standalone tool which communicates with the Test system inside the O3DE Engine, and creates its reports.

![Orchestration](orchestration.png?raw=true)

The image above shows a zoomed in data flow of the previous overview image, with more available tests. While many tests are available, only one test is being requested to execute.

The internal SystemComponent interface manages the registered mapping of available test components, as well as executing each in turn. The external commandline tool sends the parameters of which tests to execute.

The method of sending test parameters needs to be determined. While it would be possible to build the CLI tool to only perform a pass-through of launch arguments to the O3DE application, it is recommended that the CLI communicate with the SystemComponent interactively over a socket. This is mentioned below in "Open Questions"

### Test Components

Individual user-defined test components serve as containers for the logic in each test: to setup scenarios, to run test-steps, and to clean up afterward. Along each step, they will report their progress to the test-system Interface. Primary pass/fail information should be communicated to the SystemComponent from the user-defined code, following an Assert/Expect format. Everything at this layer will need to be plumbed to each supported programming language.

![Components](components.png?raw=true)

The image above focuses on the data flow between the user on the commandline, the Test Harness system, and an individual test component. To communicate pass/fail status, tests will use the same type of Expect and Assert functionality common in frameworks such as GoogleTest. Expect would verify a statement and then log the location of success/failure with the SystemComponent. Assert does the same, and then also immediately ends the test (starting its cleanup steps early).

Test assertions will need to obtain data from the systems under test. This data cannot be provided by the test framework and must be exposed by each system under test. Example tests will demonstrate patterns of exposing data to tests, such as:

- Writing a black-box test against an existing interface
  - Exposing new data through an existing interface (and when not to)
- Registering a log message listener, and adding logging statements (and when not to)
- Registering a "spy" test-double that listens to AZ::Event or EBus messages
- Mocking dependencies by modifying AZ::Interface or EBus registration

Tests existing as components attached game entities simplifies test authoring by reusing existing Editor UX. Test components attached to an entity make it easy for a test to refer to a primary owning entity. This enables easily modifying object references for tests, as well as easily moving a test's location within the game world. Certain tests may also be reusable between levels or within the same level by copy-paste.

Test names are important for legible reports, however the "same" test should still become a new unique test when copied elsewhere. To help ensure test uniqueness and enforce the order of execution and reporting, test Components should not rely solely on user-provided names or labels. Tests should include a UUID which is regenerated on copy, such as when a user uses the editor to copy-paste an entity which contains a test component.

### Report Formatting

The test report generation tools of PyTest can be reused, to create reports in the standard JUnit/xUnit XML format recognized by most test-runner tools and CI systems. Reusing python-based tools also allows reuse of existing crash watchdog fixtures already developed for LyTestTools and EditorTest. Test failure reports should note crash logs, when present.

### Filtering and Disabling Tests

Filtering should be available based on test name or UUID, and also for optional secondary tags which can label and categorize tests. Tests should also be able to flag themself as disabled, such that they do not execute even when selected to run. This can be handled with a reserved label of "Disabled" or "Skip".

To implement this, the external CLI tool could reflect test names as PyTest names, connect UUIDs and labels to PyTest marks, and then reuse the filtering and reporting interface provided by PyTest.

#### Disabling the Test System

Pre-release test code should not bloat the customer's final release product. When the Test Harness Gem gets disabled, neither the Test Harness system nor any user-defined C++ test code should get compiled into the executable.

Customers may also desire a package management tool, to prevent deploying extra unused files such as scripts, config files, and assets. However as such a tool has ramifications beyond tests, it would be appropriate to be defined by SIG-Content or SIG-Release.

## What are the advantages of the feature?

- Enables writing automated tests that run in the O3DE Engine
  - O3DE contributors can automatically verify features bundled with the product
  - Tests inside the Engine should execute faster without the Editor
  - Customers of O3DE can automatically verify their own new features
    - Tests are easily disabled from the release product
  - Tests can be configured to emit performance metrics from the Game, Server, or Editor
  - Tests can execute on mobile devices and consoles

## What are the disadvantages of the feature?

- Creates another large framework with multiple features to maintain.
- EditorPythonBindings (EPB) already allows for self-tests of O3DE, which already provides an easy way for tests to not exist in the release game. Customers *could* expose their features to this interface to write tests targeting their in-game logic.
  - However EPB feature can only be used within the context of the Editor, and not on mobile or console devices.
  - Python and the BehaviorContext used by EPB are relatively slow, and the Editor also creates performance overhead.
- May be difficult to write tests for any user-defined systems which do not use AZ::Interface or EBus.
  - Users' test code will need to access their systems.
  - Some interfaces can be intentionally immutable or inaccessible, for concerns including performance and licensing.
- Enables writing wide non-unit-scope tests, which can be slow and are often not a high return on investment.
  - Documentation can mitigate this by recommending why and where to write which type of test.
    - While often important, automated integration tests are not a "silver bullet" solution and should exist alongside other types of tests.
- Optional security features proposed, prioritized for later
  - Often means no security enabled

## How will this be implemented or integrated into the O3DE environment?

A Gem will be added to O3DE, and enabled for the AutomatedTesting project. The AutomatedTesting project should contain at least one example test. Customers can opt into enabling the Gem for their own projects. No additional dependencies are required beyond what already ships with O3DE.

## Are there any alternatives to this feature?

- The test results interface could be limited to matching log messages, with contributors sending logs directly from gameplay code
  - Lighter weight, only depends on remote command interface and logging
  - Logs have some inherent instability, and customers may disable logging for performance reasons
  - Does not simplify how users author tests
    - Easier to accidentally leave test code in final builds
- Tests could be written in C++ extending from GoogleTest up into the engine
  - Would require writing custom bootstrapping logic (per Project) and modifying the game executables (per OS)
  - Only provides C++ as a test writing language
    - Could define a pattern to trigger scripts in Lua or ScriptCanvas and report back to C++ test asserts
- Editor Python Bindings could be extended from Editor-only into the engine runtime, effectively enabling Python as a in-game scripting language.
  - Python is a relatively slow and heavyweight language, and would be difficult to deploy for use on mobile and console devices.
  - Only provides Python as a test writing language.
    - Could provide a pattern to trigger scripts in Lua or ScriptCanvas and report to Python.
- Implement less than the three proposed test-writing languages (C++, Lua, ScriptCanvas)
  - Ignoring a preferred language will reduce the potential user base interested in using this feature
    - C++ tests can be relatively performant, and easier to attach to lower-level inspection
    - Lua and ScriptCanavs tests do not require recompilation, and can be more accessible to many developers
- Only implement commandline test execcution, no Editor UI features
  - Significantly reduces iteration speed for creating level-dependent tests
  - Reduces discoverability, reducing the number of users who will try to run and author tests
- Do not implement this, and leave it to contributors and customers to build what they want
  - Writing in-game automation will continue to have a high startup cost
    - O3DE more likely to follow its current inertia of few automated in-game tests
    - Teams using O3DE will be less likely to invest in their own automated in-game tests

## How will users learn this feature?

Users will primarily encounter this feature in 4 locations:

1. Overview and Getting Started pages in the O3DE.org documentation
2. Example tests in O3DE's AutomatedTesting project, with inline "how and why" documentation
3. Elements in the Editor UI (after enabling the Gem)
4. The CLI tool to launch tests, which includes help information

## Are there any open questions?

### Interactive Socket Interface

Should an interactive socket be used, or are CLI launch arguments sufficient?

Using sockets would simplify writing networked test scenarios which sequentially step through commands between multiple devices. In the diagram below sockets could follow either orchestration-script writing pattern, whereas CLI parameters could only use the pattern on the right:

![Networked scripts](netscript.png?raw=true)

If a socket connection is not used, it will be more difficult to coordinate tests and report results. Not using a socket will also require any test filtering and iteration occur inside the Engine, increasing the complexity of what must run in each application.

The need to frequently relaunch the application with new CLI Args could be mitigated by having the EditorSystemComponent save a manifest for which tests exist. Without the manifest, even just listing out which tests are available would require the application be started and a specific level loaded (potentially every level).

#### Interactive Socket Pros

- Reduces need to re-launch the application to run multiple tests
  - CLI arg buffer has size limits dependent on OS/target settings, which can impact large command strings
- Sending test status messages over a socket reduces reliance on inspecting log files
  - Mobile device logs can be difficult to interact with
- Can provide a secondary health check to help detect if the application has become nonresponsive
- Easier to write scripts for networked tests

#### Interactive Socket Cons

- Slightly more complex to implement and maintain
- Larger security footprint

#### Remote Console

The Remote Console interface and CVars appear to already provide an interactive communication channel. Is RC deprecated? Should it be reused?

### Removing files from deployable release builds

- Is this feature necessary for a test framework?
  - Should SIG-Content or SIG-Release own defining and maintaining this mechanism?
- For ease of exclusion, should we recommend users compile their C++ tests into separate libraries, similar to existing C++ tests using GoogleTest?
  - Would do little to simplify on platforms which require Monolithic builds.

### Component Uniqueness

- Can the "same" component exist between different levels, or are they functionally copies?
  - Should a test ever be considered be the exact same test, if their environment is different?
- What additional data is required for tracking test uniqueness in a debuggable, accessible way?
  - What UX immediately gets the developer from a report to the test file?

### Gem Name

This feature needs an accessible, accurate, stable name for its Gem. The name should stay applicable even as other tools and test-features are enabled. The following names are proposed, but feedback is encouraged:

- EngineTestHarness
- TestHarness
- GameTest
- EngineTest

### Multiple Gems

Would splitting this feature across multiple gems to simplify how users select enabling features? Or would a single gem with automatic enable/disable behavior be better? (for example, what is the behavior when ScriptCanvas is disabled) The following layers could exist in separate gems:

1. Standalone Remote Tool
2. Test Harness Core (C++ interface and reporter, CTest helper, CLI runner)
3. Lua Script Extension
4. ScriptCanvas Extension
5. Editor-based test report UI

#### Reuse and Extension

Aside from modifying the code of the Test Harness System, how could someone build other test tools from the individual parts of this harness? Inside O3DE this would likely be easier with multiple Gems communicating with AZ::Events. Outside of O3DE, it would be easier to have tools exist in their own GitHub repo, with a generic event/callback mechanism.

### What UI avoids displaying too many test components?

There should be few top-level menu options to add a generic test component to an entity, after which that component can then be configured to use any available test. The list of all tests, for which a project may contain hundreds, should not exist as top-level component options in the UI. Also, C++ tests should get configured similar to script-based tests, and not as individual unique components.

### Could a test be written without opening the Editor?

Tests attached to a component can be convenient, but require modifying entities in a level. It can also be convenient to provide a way to not require hooking up to an entity in a level. How could this provide a way to define and execute tests without manually opening the editor?

### How can we minimize the overhead of using this test framework?

There are many types of overhead this feature can incur including runtime, compile time, artifacts on disk, configuration effort, and deconfigure/disable effort. Some heuristics to reduce this include:

- Prefer AZ::Interface and AZ::Event over EBus
- Incrementally report status as each test starts and completes, to avoid losing data during a crash.
- Send as little information as possible between test components and the internal results reporter.
  - Have test expectations only communicate once at the end of testing, not immediately when evaluated. (trades potential data-loss for reduced perf impact)
- Send as little information as possible from the internal System to an external report file
- Users have the option of writing a test in C++ instead of using a scripting language, which should improve runtime performance.
  - Document advice on how to write integration tests.
  - Recommend writing narrower, unit-scope tests separately in GoogleTest.

## FAQ

### Why not use GoogleTest for in-game C++ tests?

GoogleTest expects the user to invoke RUN_ALL_TESTS from the main thread, giving its singleton runner control of inspecting args, allocating its own tracking data, and running tests in its environment. There are tradeoffs to reusing GoogleTest for the type of fully-integrated tests which are common at the gameplay programming level:

#### GTest Pros

- Reuse industry standard tools already used by O3DE
- Comes with useful macros and helper functions

#### GTest Cons

- Likely need to re-engineer part of the GoogleTest runner to avoid it hogging the main thread or otherwise altering how the game runtime "normally" operates
- Likely more complex to set up GoogleTest boilerplate than to write "normal" gameplay code with a Test Harness interface
- Would require a pattern to clearly separate O3DE's existing GoogleTest unit-tests from new in-game integration tests

This deserves further investigation before delivering a C++ interface for the proposed Gem.

### Why not rely on log output for reporting test results?

The game's console logger could be directly used for test output, instead of scripts scripts routing status messages to a test-system which in turn communicates via socket.

#### Pros

- Simpler to implement
- No additional system to report test output

#### Cons

- Log statements can get lost and, unlike socket communication, missing logs are not retryable.
- While logs could get piped to a remote source to enable remote two-way communication, this will most always generate far more network traffic than a socket.
- Additional complexities for devices which are not a PC.

Execution logs remain important artifacts to collect and provide alongside test results, and are already in scope. However it is recommended that system status rely on specific and retryable signals.

### Will test start-time be synchronized and consistent between executions?

No. While it should be possible for a test to verify readiness signals before executing, there are few deterministic timing guarantees from current hardware or software. This is a general gameplay programming problem which also plagues tests. It should be adequate to investigate such issues as they appear in individual systems.
