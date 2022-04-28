# Intermittent Failure Policy

- [Intermittent Failure Policy](#intermittent-failure-policy)
  - [Introduction](#introduction)
  - [Policy](#policy)
    - [Exceptions](#exceptions)
  - [Disabling Individual Tests](#disabling-individual-tests)
    - [GoogleTest](#googletest)
    - [PyTest](#pytest)
  - [Troubleshooting](#troubleshooting)
    - [Help! Is my Test Intermittent?](#help-is-my-test-intermittent)
      - [Step 1: Identify](#step-1-identify)
      - [Step 2: Confirm](#step-2-confirm)
      - [Step 3: Document](#step-3-document)
      - [Step 4: Disable](#step-4-disable)
  - [The Sandbox Test Suite](#the-sandbox-test-suite)
  - [Further Reading](#further-reading)

## Introduction

This document describes handling unstable failures which appear to occur randomly. The most likely reason you are reading this is due to an intermittent issue in the [Automated Review Pipeline](https://jenkins.build.o3de.org/job/O3DE/).

The Automated Review (AR) pipeline performs builds and tests, as well as additional configuration and asset processing steps. Pull Requests are required to be verified by Automated Review before changes get merged. This Continuous Integration system also executes "branch update jobs" after merges, to detect complex merge-order issues which can occur in the time between verification and merge. A separate pipeline also executes at a slower cadence, which runs a wider array of periodic tests and build variants. [SIG-Build](https://github.com/o3de/sig-build/) owns the Jenkins service and scripts, and can advise on build failures. [SIG-Testing](https://github.com/o3de/sig-build/) owns test-tools as well as sanity-tests in this pipeline, which make sure the tools stay functional. While these SIGs maintain the foundation for testing, the majority of tests executed in this pipeline are owned by other SIGs. These tests, along with the applications they target, must operate deterministically.

If you are blocked, you own fixing what blocks you and/or finding support. Even if you are not blocked, the community will appreciate fixes or feedback submitted to the appropriate SIG. Which SIG owns a failure is not always obvious, as the Automated Review pipeline executes code maintained by every SIG in O3DE. If you have questions, reach out over [Discord](https://discord.gg/p3padwr58u)!

## Policy

No intermittently failing test or feature is allowed, with extremely limited exceptions. If you see a failure that appears nondeterministic, immediately remove it following these steps:

1. Determine if the failure is intermittent
   - If possible, execute on your local machine multiple times to tally a failure rate, and attempt to debug the issue
   - Execute the Jenkins pipeline no more than 10 times, saving the first failing log into a text file
   - If the failure occurs consistently ***DO NOT CONTINUE*** following these steps, and instead reach out over [Discord](https://discord.gg/p3padwr58u)
2. [Cut a GitHub Issue](https://github.com/o3de/o3de/issues/new/choose) to track both fixing the underlying issue and re-enabling the test
   - Add a label for the SIG which maintains that feature 'sig/XYZ' ***OR*** use label 'needs-sig'
3. The source of intermittent failure should be immediately removed from the pipeline
   - If possible, submit a new pull request which reverts or disables the intermittent module, feature, test, or hardware-configuration

### Exceptions

The policy above applies to all code, scripts, and hardware used in the Automated Review Pipeline with the following exceptions:

- It is rarely more risky to remove an unstable critical test (or feature) than to leave it in a known intermittently failing state
  - If you have a reason to not follow this policy, please reach out to SIG-Testing on [Discord](https://discord.gg/p3padwr58u)!
- Intermittent failures ocurring on only one Operating System are still intermittent failures, though it is reasonable to conditionally disable
- Builds or tests which reliably fail at a given commit are **not** considered intermittent
  - Such failures may appear inconsistent across multiple commits, when thrashed by rapid code changes
  - While the pipeline makes consistent difficult to introduce, when they occur it is still appropriate to immediately revert or disable until fixed

## Disabling Individual Tests

To quickly disable a specific test, modify its source as follows:

### GoogleTest

Change tests of the form:

```cpp
TEST(MyTestClassName, MyTestUnitOfWork_NotableContext_ExpectedResult)

TEST_F(MyTestFixutreName, MyTestUnitOfWork_NotableContext_ExpectedResult)

TEST_P(MyTestParameterizedFixutreName, MyTestUnitOfWork_NotableContext_ExpectedResult)
```

...to add `DISABLED` to their name:

```cpp
TEST(MyTestClassName, DISABLED_MyTestUnitOfWork_NotableContext_ExpectedResult)

TEST_F(MyTestFixutreName, DISABLED_MyTestUnitOfWork_NotableContext_ExpectedResult)

TEST_P(MyTestParameterizedFixutreName, DISABLED_MyTestUnitOfWork_NotableContext_ExpectedResult)
```

### PyTest

Change any test:

```python
def test_MyTestUnitOfWork_NotableContext_ExpectedResult():
```

...to add a [skip marker](https://docs.pytest.org/en/latest/how-to/skipping.html) to the test:

```python
@pytest.mark.skip(reason="reason this is disabled, such as an issue tracked in GitHub")
def test_MyTestUnitOfWork_NotableContext_ExpectedResult():
```

## Troubleshooting

### Help! Is my Test Intermittent?

Sometimes it is unclear whether a failure is intermittent. It can also be unclear if the test is detecting an intermittent bug, or if the test itself creates an intermittent result. Here are recommended steps to clarify and isolate the failure:

#### Step 1: Identify

First clarify what failed by reading through execution logs to find errors. If the failure occurred in Jenkins, open the BlueOcean UI and click on the red nodes. If you are executing from a command line interface you may want to save output to a file using the pipe `|` operator:
`cmake --build build/windows_vs2019 --target Editor --config profile -- /m | build_output.txt`
`ctest -C profile -L SUITE_main --no-tests=error | test_output.txt`

If the failure occurred in a build, there should be compiler or linker errors displayed.

If a test has failed, there should be a re-run command listed in the output to execute only the failed tests.

#### Step 2: Confirm

If you suspect this failure is intermittent, gather more data to prove and clarify its intermittent behavior:

- Clean your workspace, to test for incremental cache issues, with a command such as `git clean -df`
- Execute the failing step at least five times to measure how often it fails
- Isolate a single test using [GoogleTest filtering](https://github.com/google/googletest/blob/master/docs/advanced.md#running-a-subset-of-the-tests) or [PyTest filtering](https://docs.pytest.org/en/6.2.x/usage.html#specifying-tests-selecting-tests)
- For more data on order-dependent failures, randomize the order of the tests with [GoogleTest shuffle](https://github.com/google/googletest/blob/master/docs/advanced.md#shuffling-the-tests) or [PyTest-random-order](https://github.com/jbasko/pytest-random-order)

#### Step 3: Document

After clarifying the failure reason and reproduction rate, create a new [GitHub Issue](https://github.com/o3de/o3de/issues/new/choose) to track a fix. Add a label for the SIG which maintains that feature 'sig/XYZ' ***OR*** use label 'needs-sig'. To help raise awareness and priority on the issue, also reach out over [Discord](https://discord.gg/p3padwr58u)

#### Step 4: Disable

If possible, immediately disable the test from executing and/or disable the intermittent feature. This will prevent the failure from impacting all other users of O3DE.

## The Sandbox Test Suite

While fixing a stabilty issue, the test can be onboarded to the Sandbox suite. The intent of this suite is to help collect data on whether a fix has stabilized a test, and to allow submitting certain fixes before re-onboarding a test into its suite. This suite runs alongside the Periodic test suite, and does not execute in the Automated Review presubmit pipeline nor in the Branch Update postsubmit runs.

Failures in this suite are 'expected' and are only used to collect data, not to prevent a regression of functionality. This data should be informing an in-progress fix, to evaluate whether additional work needs to be done. Tests left active in the Sandbox suite for longer than one month will be disabled to save on infrastructure costs.

For more information on changing suite registration, please refer to the [O3DE Test Getting Started Guide](https://www.o3de.org/docs/user-guide/testing/getting-started/)

## Further Reading

* [Martin Fowler on Nondeterminism](https://martinfowler.com/articles/nonDeterminism.html)
* [Flaky Tests at Google](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
* [Where Flaky Tests Originate at Google](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
* University of Illinois whitepapers on [Classifying Flaky Tests](http://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf), [Flaky Test Detection](http://mir.cs.illinois.edu/marinov/publications/LamETAL20LongitudinalFlakyTests.pdf), and [Rerunning Flaky Tests](http://mir.cs.illinois.edu/marinov/publications/LamETAL20ManyFlakyTestsAreNDOD.pdf)
