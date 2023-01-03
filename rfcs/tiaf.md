## Summary

This feature proposes a change based testing system called the *Test Impact Analysis Framework* (**TIAF**) that utilizes test impact analysis (**TIA**) for C++ and Python tests to replace the existing _CTest_ system for test running and reporting in automated review (**AR**). Rather than running all of the tests in the suite for each **AR** run, this system uses the source changes in the pull request (**PR**) to select only the tests deemed relevant to those changes. By running only the relevant subset of tests for a given set of source changes, this system will reduce the amount of time spent in **AR** running tests and reduce the frequency at which flaky tests unrelated to the source changes can block **PRs** from being merged.

## What is the relevance of this feature?

As of the time of writing, it takes on average three attempts at **AR** for a given **PR** to successfully pass prior to merging. During each run, it takes on average 15 to 20 minutes for all of the C++ and Python tests to be run. When a given **AR** run fails, sometimes this will be due to pipeline and build issues (beyond the scope of this RFC) and other times due to flaky failing tests that are not relevant to source changes. This system will reduce the amount of time spent in **AR** by reducing the number of tests being run for a given set of source changes as well as reducing the likelihood of irrelevant, flaky tests from causing false negative **AR** run failures.

## Feature design description

**TIA** works on the principle that for a given set of incoming source changes to be merged into the repository, the tests that do not cover the impacted source changes are wasted as they are incapable of detecting new bugs and regression in those source changes. Likewise, the opposite is true: the tests that do cover those changes are the relevant subset of tests we should select to run in order to detect new bugs and regression in those incoming source changes.

![image](https://user-images.githubusercontent.com/77458631/200305259-29fb26d7-b566-4b0d-962d-6a9d1a518d3c.png)

In order to determine which tests are pertinent to a given set of incoming source changes we need to run our tests with instrumentation to determine the coverage of those tests so we can build up a map of tests to the production sources they cover (and vice versa). Each time we select a subset of tests we deem relevant to our incoming source changes we need to also run those tests with said instrumentation so we can update our coverage map with the latest coverage data. This continuous cycle of select tests → run tests with instrumentation → update coverage map for each set of source changes coming into the repository is the beating heart of **TIA**.

### Test selection

The circle below represents the total set of tests in our repository:

![image](https://user-images.githubusercontent.com/77458631/200312426-c654186c-eb96-475f-aead-f18ba21aa830.png)
The red circle below represents the relevant subset of tests selected for a given set of incoming source changes:

![image](https://user-images.githubusercontent.com/77458631/200312428-12c210b6-33da-4cfe-b035-bc150f752710.png)

In this example, the red circle is notably smaller than the white circle so in theory we can speed up the test run time for our incoming source changes by only running the tests in the red circle that we have deemed relevant to those changes.

### In practice

Unfortunately, _OpenCppCoverage_, the open source, toolchain independent coverage tool available on the Windows platform is far too slow for use in the standard implementation of **TIA**. Furthermore, there is no tooling available to perform TIA for our Python tests. As such, two new algorithms have been developed to speed up **TIA** enough for real world use in _O3DE_ for both our C++ and Python tests.

## Technical design description

### C++ tests

Fast Test Impact Analysis (**FTIA**) is an algorithm developed as part of the **TIAF** that extends the concept of **TIA** to use cases where the time cost of instrumenting tests to gather their coverage data is prohibitively expensive.

**FTIA** works on the principle that if we were to generate an approximation of test coverage that is a superset of the actual coverage then we can play as fast and loose with our approximation as necessary to speed up the instrumentation so long as that approximation is a superset of the actual coverage.

#### Test selection

The yellow circle below represents the approximation of relevant subset of C++ tests in a debug build for a given set of incoming source changes:

![image](https://user-images.githubusercontent.com/77458631/200316418-8b3fb647-29fd-4e76-8f70-d22a3e2318cc.png)
Notice that the yellow circle is notably larger than the red circle. However, notice that it is also notably smaller than the white circle. 

#### Test instrumentation

We achieve this by skipping the breakpoint placing stage of _OpenCppCoverage_. In effect, this acts as a means to enumerate all of the sources used to build the test target binary and any compile time and runtime build targets it depends on. The end result is that it runs anywhere from *10 to* *100 times* faster than vanilla _OpenCppCoverage_, depending on the source file count. Not only that but with some further optimizations (unrelated to instrumentation) we can bring the speed of running instrumented test targets to be at parity with those without instrumentation.

#### Is this not the same as static analysis of which sources comprise each build target?

Static analysis of the build system by walking build target dependency graphs to determine which test targets cover which production targets is another technique for test selection in the same family as **TIA** but with one important detail: it is incapable of determining runtime dependencies (e.g. child processes launched and runtime-linked shared libraries loaded by the test target, which some build targets in _O3DE_ do). Not only that but with **FTIA** we get another optimization priced in for free: for profile builds, the number of sources used to build the targets in _O3DE_ is typically considerably less than those of a debug build (the latter being somewhat a reflection of the sources specified for the build targets in _CMake_). This is due to the compiler being able to optimize away dead code and irrelevant dependencies etc., something that would be functionally impossible to do with static analysis alone.

The blue circle below represents the approximation of relevant subset of C++ tests in a profile build for a given set of incoming source changes:

![image](https://user-images.githubusercontent.com/77458631/200316418-8b3fb647-29fd-4e76-8f70-d22a3e2318cc.png)

Notice that the blue circle is notably smaller than the yellow circle and approaching a more accurate approximation of the red circle. In effect, we had to do nothing other than change our build configuration to leverage this performance optimization.

### Python tests

Indirect Source Coverage (**ISC**) is the novel technique developed to integrate our Python end-to-end tests into **TIAF**. The goal of **ISC** is to infer which dependencies a given test has in order to facilitate the ability to map changes to these dependencies to their dependent tests. Such coverage is used to associate the likes of assets and Gems to Python tests, shaders and materials to Atom tests, and so on. Whilst the technique described in this RFC is general, the specific focus is on coverage for Python tests such that they can be integrated into **TIAF**.

#### Basic approach

The techniques for inferring source changes to C++ tests are not applicable to Python tests as **FTIA** would produce far too much noise to allow for pinpointed test selection and prioritization. Instead, Python test coverage enumerates at edit time components on any activated entities as their non-source coverage to determine which Python tests can be selected when the source files of these components are created, modified or deleted.

An additional optimization step is performed: for each known component, the build system is queried for any other build targets (and their dependencies) that any of the known component's parent build targets are an exclusive depender for. These exclusive dependers are thus also deemed to be known dependencies for any Python tests that depend on said parent build target.

#### Differences Between C++ Coverage and Test Selection

Python test selection differs from C++ test selection on a fundamental level: C++ test selection is additive, whereupon the selector optimistically assumes NO C++ tests should be unless a given test can conclusively be determined through analysis of coverage data whereas Python test selection is subtractive, whereupon the selector pessimistically assumes ALL Python tests should be run unless a given test can be conclusively determined through analysis of non-source coverage. This subtractive process allows for test selection to still occur with only tentative, less rigorous coverage data that can be expanded upon as more rules and non-source coverage options are explored without the risk of erroneously eliminating Python tests due to incomplete assumptions and limited data.

#### Limitations of **ISC**

The limitations of this proposed approach are as follows:

1. Only Python tests that have component dependencies are eligible for test selection.
   a. All other Python tests must be run.
2. Only change lists exclusively containing CRUD operations for files belonging to the build targets for components (or their exclusive dependencies) that one or more Python test depends on are eligible for test selection.
    b. Otherwise, all Python tests must be run.

#### Technical Design

![image](https://user-images.githubusercontent.com/77458631/200323935-8c7be7cd-ef5d-4b80-b947-9d4acf54f86b.png)

The _PythonCoverage_ Gem is a system Gem enabled for the _AutomatedTesting_ project that enumerates at run-time all components on all activated entities and writes the parent module of each component to a coverage file for each Python test case. Using this coverage data, the build target for each parent module binary is inferred and thus the sources that compose that build target. Should a change list exclusively contain source files belonging to any of the enumerated components it is eligible for Python test selection. Otherwise, it is not an all Python tests must be run.

##### Coverage Generation

1. The Editor is launched with the Python test being run supplied as a command line argument.
2. A _OnEntityActivated_ notification is received for an entity that has been activated.
3. The component descriptors for the recently activated entity are enumerated.
4. For each enumerated component descriptor, an attempt is made to discover the parent module that the descriptor belongs to.
    a. As components belonging to static libraries do not have a parent module, only those belonging to shared libraries (i.e. Gems) can be used for coverage.
5. The parent modules discovered for the Python test are written out to a coverage file for that test.

##### Dynamic Dependency Map Integration

1. For each Python test coverage file, the build system is queried to find the build target for parent modules in that coverage file.
   a. Determine the modules that this module is an exclusive depender for.
2. The source files for each build target are added to the Dynamic Dependency Map as being covered by the Python test(s) in question.

##### Test Selection

1. The change list is analysed to see if it exclusively contains source files for known Python test dependencies:
   a. If yes, the Python tests dependent on those dependencies are selected for the test run (Python tests not dependent are skipped).
   b. If no, all Python tests are run.

## What are the advantages of the feature?

For the most part, the benefits will come without any need for input from team members as it will be running as part of **AR** to reduce the time your **PRs** spend in **AR**, as well as reducing the likelihood of you being blocked by flaky tests. However, the framework itself is a complete model of the build system so there is a lot of scope for using this information to implement even more **AR** optimizations. For example, as the framework knows exactly which sources belong to which build targets, and which of those build targets are test targets and production targets, it knows that, say, a text file is not part of the build system (or covered by any tests) so changes to that file can be safely ignored by **AR**. Although it’s hard to speculate what the long term benefits of this framework will be, we are confident that other teams and customers will make good use of this framework to implement all manner of novel optimizations and useful repository insights.

### C++ tests

On average, we are currently rejecting 85% of tests to run for incoming source changes which has resulted in speedups of approximately 5x compared to that of running all of our tests (and there’s still a lot of room for improvement, an end goal of >95% test rejection and 15-25x overall speedup of test run times is realistic once all the other pieces are in place).

![image](https://user-images.githubusercontent.com/77458631/200341507-d3ee5274-1789-4dd8-9e0b-dc78d5c16e6a.png)

Above is the plot of test run duration for 130 builds in O3DE's AR servers using **FTIA**. The orange line represents the average duration of all of thees builds (which currently stands at just over 30 seconds). Notice that many builds take significantly less than this, and it is only the long pole test runs (those where the source changes touch root dependencies, e.g. _AzCore_) that are pushing the average higher.

![image](https://user-images.githubusercontent.com/77458631/200341538-12efedf3-9010-47f4-82cc-5998c689e892.png)

Above is the selection efficiency (amount of tests rejected as being irrelevant for the source changes) for the same 130 builds. The average efficiency is about 85% (85% of tests being rejected) with many more being at or near 100%.

### Python tests

We ran the Python test selector for the past 50 pull requests and measured the test selection efficiency, that is, the percentage of the 26 Python test targets in the Main suite that were rejected as being not relevant to the source changes in the PR. Our baseline efficiency to beat, as offered by our current _CTest_ approach, is 0% (no test targets rejected), so anything above 0% would be considered a win.

![image](https://user-images.githubusercontent.com/77458631/200358532-4092f865-504d-4d4b-8971-fa1d31363cb4.png)

In the graph above, the orange line represents the average test selection efficiency over all 50 **PRs** (currently sitting at 30%). That is to say that, on average, 30% of test targets are rejected for a given **PR** (even if the data itself is very polarized). Although this falls far short of the 85%+ efficiency of the C++ test selection under Fast Test Impact Analysis, even if no further improvements were to made we would still be cutting down the number of Python tests our customers need to run in AR by nearly a third. Over time, these numbers add up to significant, measurable savings in terms of server costs and velocity that **PRs** can be merged into the repository. Of course, we do have plans to improve the efficiency, but that’s a challenge for another day.

## What are the disadvantages of the feature?

This new system undoubtedly introduces far more complexity than using the existing _CTest_ system. This complexity in turn means responsibility to further develop and maintain the system, as well as more points of failure that could hinder **AR** should any breaking changes be introduced into the system.

## How will this be implemented or integrated into the O3DE environment?
The average developer need do nothing but sit back and reap the benefits of time savings in **AR** when they wish to merge their **PRs**. For the teams responsible for maintaining the build system and AR pipeline, this system will require the _CTest_ pipeline stages to be replaced with the **TIAF** pipeline stages. We have been testing this system with our own shadow pipeline to weed out any teething issues so will be able to provide a strategy for seamless replacement into _O3DE_'s **AR** pipeline.

Currently, only Windows is supported. Should this system be rolled out with success, other platforms will be supported.

## Are there any alternatives to this feature?
There are no free, open source and toolchain independent tools for providing **TIA** on the Windows platform.

## How will users learn this feature?
Documentation will be provided but, for the most part, this feature will be transparent to the user. There is nothing they need to do in order to use this system other than open up a PR.

## Are there any open questions?
* How will other platforms be supported?
* How can this system be integrated with the least amount of friction and disruption?
* What are the unforeseen blind spots in coverage for both C++ and Python tests?