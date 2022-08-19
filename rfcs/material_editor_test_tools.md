# RFC: Extend EditorTest tools to MaterialEditor

## Summary:

There are a set of tools inside `C:\git\o3de\Tools\LyTestTools\ly_test_tools\o3de\editor_test.py` that allow for parallel test runs that run in batched sets. The problem is, these tools only work for tests that utilize Editor but it could also support MaterialEditor. It currently cannot support other executables (such as Launcher executables).

This RFC proposal will discuss the viability of this and also a rough outline of how we would go about implementing it for O3DE. Once completed, the results of this RFC will be used to cut tasks and assign work to get it implemented.

## What is the relevance of this feature?
Since the inception of the parallel & batch test run tools, there has been an increase in test speed without much sacrifice to test efficiency. This is because these tools utilize return codes to validate test results instead of log lines, which are often plagued by race conditions, file permission issues, and other technical problems whereas return codes are not.

This change will expand this option beyond just tests that utilize Editor and apply it to the MaterialEditor also.

## Feature design description:
The feature will be the expansion of our existing parallel & batched test tools. Once implemented, it would allow the existing parallel & batched test runs for Editor.exe to be extended to MaterialEditor test runs as well, utilizing this return code approach to verifying tests. We will save time on test runs while also gaining more test efficiency, with no real sacrifice to our existing approach to automated tests.

## Technical design description:
It will work the same as Editor tests that utilize this functionality. There is a "test launcher" file that contains all of the tests to batch, run in parallel, or run as a single non-batched non-parallel test. An example launcher file would look something like this:
```
# Full example can be found at C:\git\o3de\AutomatedTesting\Gem\PythonTests\Atom\TestSuite_Main.py

from ly_test_tools.o3de.editor_test import EditorSharedTest, EditorTestSuite

logger = logging.getLogger(__name__)
TEST_DIRECTORY = os.path.join(os.path.dirname(__file__), "tests")


@pytest.mark.parametrize("project", ["AutomatedTesting"])
@pytest.mark.parametrize("launcher_platform", ['windows_editor'])
class TestAutomation(EditorTestSuite):

    enable_prefab_system = True

    @pytest.mark.test_case_id("C36529679")
    class AtomLevelLoadTest_Editor(EditorSharedTest):
        from Atom.tests import hydra_Atom_LevelLoadTest as test_module

    @pytest.mark.test_case_id("C36525657")
    class AtomEditorComponents_BloomAdded(EditorSharedTest):
        from Atom.tests import hydra_AtomEditorComponents_BloomAdded as test_module

    @pytest.mark.test_case_id("C32078118")
    class AtomEditorComponents_DecalAdded(EditorSharedTest):
        from Atom.tests import hydra_AtomEditorComponents_DecalAdded as test_module

```

The above example would run 3 tests in parallel, and as 1 batch (batch limits are 8 tests at a time, which can be changed inside the `C:\git\o3de\Tools\LyTestTools\ly_test_tools\o3de\editor_test.py` file but AR will assume this value of 8):
```
    # Function to calculate number of editors to run in parallel, this can be overriden by the user
    @staticmethod
    def get_number_parallel_editors():
        return 8
```

Since it batches the tests up using Editor as its base, all of the classes are "EditorX" classes where X is the additional naming for the class (i.e. `class EditorTestSuite`). These classes also define values that are closely tied to the Editor meaning that we should be able to implement this for MaterialEditor very easily by simply using the same objects and then making sure it targets MaterialEditor instead of Editor when launching the tests:
1. Find the entry point where "Editor" is launched in the code then provide the user an option to utilize "MaterialEditor" instead. This appears to be primarily in the class `EditorTestClass.collect().make_test_func()` method or `_run_parallel_batched_tests` function, specifically the `editor` parameter would need to reference MaterialEditor instead. This `editor` parameter feeds into all of the other function calls within the script, so will work seamlessly with MaterialEditor once the switch is made.
2. Update all Editor hard coded references such as `editor.log` to change based on the executable used by the test.
3. Add some feature optimizations to match the options available to us, such as changing `use_null_renderer` to something like `use_rhi` and then pass in the options `null`, `dx12`, or `vulkan` for instance. Basically updating the code to  be more agnostic so that it supports both Editor and MaterialEditor options rather than fit a narrowly defined run for non-GPU null renderer tests.

Also, not every test has to be run in parallel and batched. In fact, GPU tests have to be run as a single test using the `class EditorSingleTest` object so this option will exist for any tests that can't be run in parallel for whatever reason. The added benefit of converting is primarily for the return codes which are the most solid test approach (previously we used log lines).

Initial support of this feature would be only for Windows, as that is our current standard. This could change in the future, but this RFC will not cover any of that design.

## What are the advantages of the feature?
Faster test runs, more test stability (return codes instead of log lines), and easier than before to add new tests to an existing python test launcher script. All existing tests are also ready today to be converted in this way (and many have already).

## What are the disadvantages of the feature?
The disadvantage will be that some unknowns remain as to how easy this "switch flip" would be from Editor to MaterialEditor - it looks easy to do when reviewing the code, but will it be easy when we change it and try to run MaterialEditor tests in this manner? That will be the large unknown this task work would unveil for us - though I predict it being workable.

## How will this be implemented or integrated into the O3DE environment?
Very easily by simply adding to the existing code inside `C:\git\o3de\Tools\LyTestTools\ly_test_tools\o3de\editor_test.py` or the `C:\git\o3de\Tools\LyTestTools\ly_test_tools\o3de\` directory. It uses files that already exist and functionality that has already been tested with Editor.exe extensively.

The hard part will be testing, hardening, and getting other users to adopt and update their code to match this new functionality. This would ideally be done using an e-mail linking to a wiki that details for users how to update their tests for this framework change so that nothing breaks unexpectedly (especially when it comes to renames, they're trivial but they can easily break a lot of existing code).

## Are there any alternatives to this feature?
The alternative we had were log line tests which are not as reliable as return code tests.

## How will users learn this feature?
There is already an existing internal wiki, and we would probably have need of a public one since this will be publicly available. However, since the demand for test tools has appeared low and most of our sig-testing meetings are usually only internal participants, I don't think a public wiki would be required initial - though it certainly wouldn't hurt in the long run.

## Are there any open questions?
An open question is whether GPU tests will ever be able to run as parallel & batched tests. So far it looks like the answer is no, but these types of tests are often cumbersome in automated testing so are usually avoided or greatly reduced in volume compared to other types of tests since they are considered "front end tests". "Front end tests" are usually the slowest and most prone to race conditions, so even though this is a problem I believe it will always be one regardless of which automated test tool we choose.
