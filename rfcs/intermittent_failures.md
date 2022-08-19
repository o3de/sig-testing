# RFC: Intermittent Failure Intervention

## Summary

The Testing Special Interest Group (SIG-Testing) primarily serves a support and advisory role to other O3DE SIGs, to help them maintain their own tests. While this ownership model tends to function well, there are cases where instability in features owned by one SIG can interfere with the tests of all SIGs. This RFC proposes a runbook for SIG-Testing to follow during emergent cases where the intermittent failure rate approaches a critical level. It also proposes metrics with automated alarms that trigger proactively following this runbook on behalf of other SIGs, as well as improved automated failure warnings to all SIGs to reduce the need to manually follow the runbook:

1. Improved automated notifications should serve as the earliest warning, notifying a SIG of failures in shared branches such as Development and Stabilization
   * Automated notifications are added based on failure rate metrics, to help inform the SIG when instability is growing in code they own
2. A runbook for SIG-Testing is intended as a fallback, to help intervene when instability in one SIG's area of ownership threatens the ability for all O3DE contributors to ship code
   * Guide to stabilize or back-out code with critical instability that impacts testing (ideally before all engineering work becomes blocked)
     * SIG-Testing will temporarily take ownership to ***modify, disable, or remove*** misbehaving code and tests, then return ownership of feature functionality to the primary SIG
   * Secondary automated notification for SIG-Testing to use the runbook exist via a second, less-sensitive threshold than the initial instability notifications sent to SIGs

Note: Investigation and intervention on behalf of other SIGs is currently outside of the stated responsibilities of SIG-Testing, and accepting this RFC would amend the charter.

## What is the motivation for this suggestion?

Intermittent failures are a frustrating reality of complex software. O3DE SIGs already strive to deliver quality features, and do not intentionally merge new code that intermittently fails. And in many cases intermittently unsafe code is caught and fixed before it ships: during development, during code reviews, or by tests executed during Automated Review of pull requests. This RFC does not seek to change how code is developed, reviewed, or submitted. Regardless, some percentage of instability evades early detection and creates nondeterminism.

When a failure appears to be nondeterministic, it can initially pass the Automated Review pipeline only to later fail during verification of a future change. Since these failures can "disappear on rerun" and are easy to ignore, they tend to accumulate without being fixed. This debt of accumulated nondeterminism wastes time and hardware resources, and also frustrates contributors who investigate failures they cannot reproduce. While [a policy exists](https://github.com/o3de/sig-testing/blob/main/intermittent_failures.md) to help contributors handle intermittent failures, its guidance has proven insufficient to prevent subtle issues from accumulating into a crisis. For example, documents such as this RFC are produced every 3-6 months when a pipeline stability crisis occurs. If this RFC is accepted, whenever the rate of intermittent failure rises above a threshold, an automated notification will prompt a SIG-Testing member to follow the runbook. Such interventions have regularly been necessary in the past, but had insufficient metrics, no automation, and no runbook.

To limit the frequency a human must manually follow this runbook, SIGs should also get automatically notified of failures well before a critical failure rate is reached. [Existing autocut issues](https://github.com/o3de/o3de/issues/7795) contain little information specific to the failure, and are not deduplicated based on this information. This can be improved to cut separate issues for different failure-causes. This can additionally combine with information from GitHub Codeowners to automatically find an appropriate SIG label to assign new issues for investigation.

The intended outcome is:

1. Improved detection of individual intermittent failures
2. Clearer automated notification of accumulated failures which are similar
3. A backup plan for maintaining functionality when the outcomes above are insufficient to maintain pipeline stability

## Suggestion design description

### Definitions

Automated Review (AR): the portion of the Continuous Integration Pipeline which gates merging code from pull requests into a shared branch such as Development or Stabilization. Test failures here include intentional rejections, where the system is functioning normally and rejecting bad code, as well as unintended intermittent failures. Due to this, AR metrics are not used in the automation proposed below, though they may still be a useful health metric.

Branch Update Run: These builds are post-submission health checks, executed against the current state of the shared branch. All failures here are unintentional intermittent failures, or are a sign of a merge error. If Branch Update runs are failing then Automated Review runs (of merging in a new change) should similarly be failing. This is the primary source of health metrics proposed for this RFC.

Periodic (Nightly) Builds: These are periodic health checks, which execute a broader and slower range of tests. Health metrics should also be reported from here.

#### Tolerable Failure Rates

Metrics on test failure rates are an inherently imprecise and fuzzy measurement, which try to demonstrate statistical confidence. A single piece of code may have one extremely rare intermittent failure, or it may have multiple simultaneous patterns of intermittent failure, or it may eventually become consistently failing due to complex environmental factors. The following confidence bands intend to simplify interpreting fuzzy data, starting from the most severe:

1. Consistent: A failure is occurring so often that maintainers are effectively unable to merge code. Failures at this threshold and higher become difficult to distinguish from a 100% failure rate.
2. Severe: A failure occurs fairly consistently, creating significant increases to the time and cost to merge code. At this rate of failure, intervention becomes necessary.
3. Warning: A failure occurs inconsistently, though not frequently enough to cause severe throughput issues. At this rate of failure it becomes more important to isolate and fix an issue.
4. Detected: A failure has occurred at least once, but may be relatively rare. In this case it may be important to fix, or may be considered a non-issue. (current notifications only trigger on each detection of failure)
5. Undetected: A failure occurs in zero of the recently sampled cases. This does not mean an issue does not exist, but that it is effectively undetectable by existing automation. If a bug does exist, it could theoretically be detected by expanding the sample set.

Within these categories, some already have obvious steps to follow. Consistently failing issues will continue to follow the GitHub issues workflow. Issues undetected by automated tests either prompt new automation to detect them, or may be safe to ignore. And for the purposes of this proposal, the "Detected" category is a threshold to not require additional action beyond continuing to auto-cut an issue to notify about the failure. The boundaries between the remaining categories are proposed as:

* Severe: a failure rate between 1/20 and 1/2 (above this would be Consistent)
* Warning: a failure rate between 1/100 and 1/20 (below this would be Detected)

These thresholds are subjective and are sensitive to the scope of the product, its tests, and its pipeline environment. Due to subjectivity, the values may need to change as O3DE changes in scope. To better handle the broad scope, metrics are proposed at three aggregated levels. Each level acts as a filter with different sensitivity, catching what the previous one misses. The intent of these categories is to accurately identify a problem area when possible, but still detect when small problems accumulate into a widespread issue. To keep the definitions simple, the same confidence bands are proposed for the metrics categories. This is described below in the section "Metrics for Failure Rates".

### Autocut Issue Improvements

[Existing autocut issues rarely result in action and pile up](https://github.com/o3de/o3de/issues/7795) from failed Branch Update Runs. This proves that these issues are not effectively tracked beyond the push-notifications sent as instant messages. The runbook above calls for advance notifications sent to SIGs, and suggests the existing autocut issues are the appropriate medium. To effectively use them, the following improvements are recommended:

1. Autocut deduplication, where a preexisting issue is updated instead of cutting a new issue, needs to be based off additional failure information. Instead of targeting a single issue title, failure root cause and origin information should be included. Modifying deduplication to include failure analysis will result in dozens more autocut bugs. This analysis needs to be carefully made less-generic, as certain bugs have multiple failure patterns. Becoming overly sensitive (for example to logging statements with timestamps) could result in never deduplicating, and cutting multiple unrelated issues tracking the same bug.
2. A form of failure root cause analysis is already used within Jenkins. The output of this plugin can be used for deduplication. If there are additional patterns we want to use to dedupe issues, they can be added to the patterns used by this tool. If the output of this tool is too noisy, we can add a filter between it and the deduplication mechanism.
3. When a failure occurs that references a file of origin (compilation failure, test failure, etc.) the system can look up that file in GitHub Codeowners, and find most likely SIG to initially assign the new issue.
4. For convenience, a link will be included in the Jenkins results summary whenever any issue gets autocut/deduped from a build.

### Failure Runbook

When any pipeline failure first occurs, it will result in an autocut issue. This issue should be auto-assigned to the SIG designated by the [Codeowners file](https://github.com/o3de/o3de/blob/development/.github/CODEOWNERS) for investigation. As an issue continues to reproduce, existing issues should be auto-commented on. And if failure rates rise above a threshold, a second issue gets cut to SIG-Testing to intervene by following a runbook. The full runbook is not defined here, only an outline of how it is used.

This runbook will document both Automated and Manual processes to reduce intermittent failure. The automated processes are documented to clarify the steps that have already been taken.  When the initial automated portions are insufficient, the automation prompts SIG-Testing to take action in the manual portion of the runbook by auto-cutting an extra issue in GitHub. The following steps apply to Branch Update Runs and Periodic Builds, with the intent to keep Automated Review runs only seeing newly-introduced failures. **(After RFC, this runbook should exist as its own document in the SIG-Testing repo)**

Pipeline Automation:
When any failure is detected in a Branch Update or Periodic Build, an issue is updated or auto-cut to track it (this is already implemented today). If tests were executed, then test metrics will also be uploaded. It is expected that autocut issues which are due to intermittent behavior may be claimed and then closed due to no reproduction, or ignored due to low priority.

Issues Automation with Metrics:
When new test metrics are uploaded, any new failure should prompt querying recent failure metrics. Based on the query results, take the following actions:

1. An individual test failure is above the Warning threshold:
   * Update autocut issue with name of failing test, failure rate, and a link to the intermittent failure policy
   * If also above Critical, add label `priority/critical`
   * If also above the Consistent threshold, Autocut (or update) a second issue to SIG-Testing and relate it to the original with label `priority/critical` to investigate rising failure rates
2. A failing test-module is above the Warning threshold:
   * Update autocut issue with name of failing module, failure rate, and a link to the intermittent failure policy
   * If also above Critical, add label `priority/critical`
   * If also above the Consistent threshold, Autocut (or update) a second issue to SIG-Testing and relate it to the original with label `priority/critical` to investigate rising failure rates
3. All pipeline runs are above the Critical threshold:
   * Autocut (or update) a second issue to SIG-Testing and relate it to the original with label `priority/critical`, to investigate rising failure rates with unclear origin
   * Include a link to this SIG-Testing runbook
   * If also also above the Consistent threshold, update label `priority/blocker` on the investigation ticket

Note: Warning threshold is currently undefined at the pipeline level, as it is more sensitive to failures.
Note: Creating a pipeline critical will always occur before module or individual test critical. However it may not get investigated before other more-specific critical issues are logged. (This may result in nearly always having an investigation open)
Note: Can result in a SIG-Testing investigation being prompted on the first new failure shortly after a prolonged failure.

### Metrics for Failure Rates

Three levels of test metrics are proposed, and each require alarms that trigger automated actions in GitHub Issues.

#### Test failures per pipeline run

Contributors are most directly impacted by the aggregate test failure rate of an entire run of the Automated Review Pipeline. This involves running tests across all modules for multiple variant builds in parallel, and O3DE has grown to nearly 200 test modules. Certain modules run in parallel with one another, and certain failures may only occur when all modules execute. There are around 100,000 tests which currently execute on each pipeline run, and a single intermittent failure across any of these tests results in a failed run.

#### Failures per test module

Test modules often contain hundreds of individual tests. In a test module with a hundred tests, if every test had only a 1% independent failure rate then the module would be statistically expected to almost always fail. Additionally, certain tests may only fail when run with the rest of their module. Due to the finer granularity and scale, these metrics are less sensitive than those for the full pipeline. For instance, if each of the current ~200 modules contain only a single 1/100 error rate, barely triggering a warning-level response for modules, then across the two variant test executions the pipeline would expect a 100% failure rate with around 4 failed modules in every run. `( 2 build variants * 200 modules * 1/100 fail rate = 4 failures per pipeline run)`. While an unhealthy state is possible with a low per-module failure rate, it should still be detected by the pipeline-wide metric above.

#### Failures per individual test

Individual tests must be highly stable. They are also the smallest, quickest data point to iterate on. With nearly 50,000 (and growing) tests across the Main and Smoke suites, even a one-in-a-million failure baked into each test could accumulate into severe pipeline-level failure rates `( 2 build variants * 50,000 tests * 1/1,000,000 fail rate = 1/10 runs fail)`. While this makes subtle issues in individual tests the least sensitive to accumulated failure, identifying a single problematic test is also the best case scenario for debugging. And while these per-test metrics detect only specific issues, other complex systemic issues should be caught by the aggregate metrics. An unhealthy state is again possible with a low per-test failure rate, but should l be detected by investigations into either the module-wide or pipeline-wide metrics above.

#### Metrics Requirements

A metrics backend system needs to track historical test failure data in Branch Update Runs and Periodic Builds, which will be queried to alarm on the recent failure trends.

The following metrics need to be collected from all tests executed in every run:

* Test Name
* Pass/Fail
* Build Job ID
* Pipeline run ID

The following needs to be collected from every Test Module run by CTest in a pipeline:

* Test Module Name
* Pass/Fail
* Build Job ID
* Pipeline run ID

The following needs to be collected from every test Build Job execution:

* Build Job ID
* Pipeline run ID
* Pass/Fail (all tests passed / any failed)

The following needs to be collected from every Pipeline execution:

* Pipeline run ID
* Pass/Fail (combined across all Build Jobs in the run)

To ensure statistical accuracy, the metrics analysis should be conducted across the most recent 100 runs from within 1 week. This should provide a balance between recency and accuracy.

The heaviest of these metrics will be for individual tests in Branch Update Runs. Test name identifiers are often in excess of one hundred characters, and there are currently nearly 50,000 tests across the Smoke and Main suites. With around 12 branch update runs per day triggering two test-runs each, this can result in a sizeable amount of data. Periodic Builds currently execute a few hundred longer-running tests as often as twice per day, and would constitute less than 1% of the total data. Periodic builds may also change in frequency depending on the needs and scale of the O3DE project.

To reduce the volume of data, we can store only individual test failures and calculate passes based on total runs of a build job. This may result in builds that fail early (during machine setup, build, get aborted, etc.) artificially inflating the test pass rate, since they would not create test metrics. Newly added tests would similarly start with a inflated pass rate. Further analysis on this exists below in the Appendix on Metrics Estimates.

### Example intermittent failure scenario

An individual test "A" has already failed a few times within the previous week during Branch Update Runs, and encounters two failures during some of the Branch Update Runs today. The automation would take the following steps on as new failures occur today:

* Run 1
  * Test A fails, uploading metrics
  * Collect failure information from Jenkins "Identified Problems" root cause summary and "Test Result" summary
  * Check GitHub Issues for an open autocut failure, with a description containing this root cause summary and test name, which is found already created
  * Query the failure rate metrics for the test, which is now 4/100 and below the severe failure rate
  * Query the failure rate metrics for the test-module which is now 5/100 (due to another intermittently failing test)
    * Update the existing issue with label `priority/critical` and add a comment with the name of the failing test module and its failure rate
* Run 2
  * Passes! Metrics are uploaded, and no further action is taken
* Run 3
  * Test B fails, uploading metrics
  * Collect failure information from Jenkins "Identified Problems" root cause summary and "Test Result" summary
  * Check GitHub Issues for an open autocut failure, with a description containing this root cause summary and test name, which is *NOT* found
  * Create a new issue for the failure in Test B (which may not be intermittent)
  * Query the failure rate for failing test, which is now 1/100
  * Query the failure rate for the failing test module which is now 1/100
  * Query the overall failure rate for all Pipeline runs, which is now 5/100
    * Create a new issue with label `priority/critical` assigned to SIG-Testing to investigate increasing overall failure rate
* Run 4
  * Passes! Metrics are uploaded, and no further action is taken
* Run 5
  * Test A fails, uploading metrics
  * Collect failure information from Jenkins "Identified Problems" root cause summary and "Test Result" summary
  * Check GitHub Issues for an open autocut failure, with a description containing this root cause summary and test name, which is found (same as Run 1)
  * Query the failure rate for failing test, which is now 5/100
    * Update the existing issue with label `priority/critical` (re-added in case a user removed it) and add a comment with the name of the failing test and its failure rate

## What are the advantages of the suggestion?

* Stop more nondeterministic pipeline crises before they start.
  * Expose metrics on failure rates which other SIGs can passively monitor.
  * Actively warn other SIGs when their code is increasingly misbehaving, and they have not taken action.
  * Take action when accumulated error cannot be addressed by SIGs.
* Improved issue deduplication to help all SIGs track new issues.

## What are the disadvantages of the suggestion?

* Taking action on behalf of SIGs may act as an unintended pressure valve, reducing existing habits to take proactive action.
  * Mitigated by: improving autocut tickets to accurately notify the owning SIG.
* Runbook expands stated responsibilities of the relatively small SIG-Testing, competes with delivering and improving test automation tools.
* Expanding issue automation increases its maintenance complexity.
  * Current autocut issues are low-value, but also low-noise and low-frustration.
  * Automated creation and deduplication can never be perfectly concise and accurate, this will only shape the patterns of noise.
  * Increasing the volume of autocut issues (even with accurate deduplication) has diminishing returns on resolving more bugs.
* Does not prevent new intermittent issues from passing through Automated Review, detects failures after submission.

## Are there any alternatives to this suggestion?

1. AR Test failures can be automatically retried, bypassing current user pain points by ignoring intermittent failures.
   * Succeeding after automated retry is effectively ignoring certain negative results, and accepting increased infrastructural cost they incur.
     * Increases cost and time to run tests.
   * Does not prevent shipping intermittently failing code.
     * Increases the rate of false-pass failures propagating across branches, and then into projects which depend on O3DE.
     * A "shift-right" solution that forwards pain of intermittent product bugs toward release to the end user, where issues become more expensive to fix.
   * Metrics on actual failure rate become even more important to monitor, since many failures become automatically ignored.
   * Not a mutually exclusive solution. Could still be delivered alongside this RFC.

2. AR Test failures can be automatically retried, still failing if the initial test failed, but collecting additional testing metrics to display in AR.
   * Already is proposed in https://github.com/o3de/sig-testing/issues/22
     * Not initially pursued, primarily due to Infrastructural cost constraints of rerunning tests.
   * Does not prevent shipping intermittently failing code.
     * Helps clarify AR false-failures as intermittent.
     * Does not detect intermittent false-passes.
   * Not a mutually exclusive solution. Could still be delivered alongside this RFC.

3. Tests in Automated Review could unconditionally run multiple times, to find intermittent behavior within a single change.
   * Significantly increases the time and infrastructure cost of running Automated Review
     * Costs are multiplied when growing the test suite
   * Increases the rate that existing intermittent failures block submission in AR
     * Increased blocking pain may increase priority on resolving intermittent issues, but is not guaranteed to. May instead lead to attempts to bypass automation.

4. Periodically run tests dozens to hundreds of times, collecting failure metrics separately from AR and Branch Updates.
   * Would collect metrics identical to those proposed above for Branch Update Runs.
     * More metrics would improve statistical fidelity.
   * Creates significant Infrastructural cost.

5. Pipeline failure metrics alone could be recorded and published for other SIGs to act on how they see fit, without a failsafe process for SIG-Testing to follow.
   * Channels existing maintainer frustration toward "Automated Review" as a whole being unreliable, risks forfeiting trust in the tool.
     * Investing in this may not resolve any issues.
     * Metrics not tied to a specific policy are easy to ignore.
     * Other SIGs are not currently requesting these metrics.

6. Use this proposal, but set different stability standards for different test types. This could be between C++ and Python tests, between the Smoke and Main test suites, or another partition.
   * Does not address the overall stability issues in AR.
     * Only divides where the budget of "tolerable" instability gets spent.
   * Complex rules about "what can fail how often" obscure the actual problem: a feature or its test is too unstable to verify in Automated Review.

## What is the strategy for adoption?

* O3DE Maintainers begin receiving modified failure notifications in autocut issues.
* SIG-Testing gets cut investigation issues, scheduled as maintainers find time.
  * The runbook will contain documentation for SIG-Testing to understand the metrics which trigger using the runbook.
  * Stability investigations should be completed within one month of being cut.
*Approval of this RFC should involve input from all SIGs. Delivery of these changes may need coordination with SIG-Build.

## Are there any open questions?

* Which metrics solution do we onboard to?
  * Public O3DE metrics are currently blocked by https://github.com/o3de/sig-testing/issues/34 and may need an interim solution. However this RFC intends to clarify the metrics requirements, and have a plan ready for when they become unblocked. All data handling considerations for test-data should be handled in that issue.

## Appendix

### Metrics Estimates

Below is an rough estimate of the volume of test metrics and their cost. Other SIGs may have additional metrics needs, which are not calculated here.

#### A. Jenkins Pipeline Metrics

There are currently a total of 29 stages across the seven parallel jobs in each of the Automated Review (Pull Requests) and Branch Indexing (Merge Consistency Checks) runs. There are approximately 40 pull request runs and 20 branch updates per day (across two active branches).

Periodic Builds have many more jobs, currently around 180 stages across 46 parallel jobs. It is difficult to estimate how this set of stages will change over time.

The daily Jenkins-metrics load factor of "Pipeline Run" and "Job-Stage Run" should be around `60x29 + 180x3 = 2280`. If we remove Periodic Mac builds, this would be around `2200` Jenkins-level metrics per day. It is difficult to estimate how this set of pipelines and stages will change over time. However the top level metrics should stay a comparatively low volume.

#### B. Test Result Metrics

There are currently around 43,000 tests that run in each Automated Review and Branch Update test-job, and this is expected to slowly grow over time. One path to reducing the scope of test metrics is to bundle these metrics into only reporting on the module that contains sets of tests, for which there are currently 135 modules in Automated Review. This is a major tradeoff of data quality for a ~99.9% reduction in size, which should at least be paired with saving the raw number of pass, fail, error, and skipped tests.

Another way to reduce metrics is to not explicitly store test-pass data, and to add entries for only non-pass results. This has the negative effect of conflating (reconstructed) data on "pass" and "not run" and makes it unclear when a test becomes renamed or disabled, but otherwise stores explicit failure data with significantly reduced load. This would result in a variable load of metrics which increases as more tests fail per run. Currently around 1/10 of test runs encounter a test failure. When such failures occur, the current average number of failures is around 2.5.

The daily load factor for all test-metrics would be `43,000x2x60 + 2,000x4x3 = 5,184,000` test-level metrics per day.

If only modules are reported, this would be `135x2x60 + 34x4x3 = 16,608` module-level metrics per day.

If modules and test-failures are reported, this would be approximately `0.1x2.5x60 + 135x2x60 + 34x4x3 ~= 16,625` module-plus-failure metrics per day. Since this load is variable, it is rounded up to `20,000`. This suggests that saving only failure data would be a ~99.6% reduction in storage.

#### C. Profiling Metrics

There are currently 1795 Micro-Benchmark metrics (across 10 modules), and 10 planned end-to-end benchmarks of workflows. Providing a metrics pipeline will encourage the current number of performance metrics to grow, as such profiling data otherwise has little utility. A wild estimate is that this will expand by 10x within a year.

These execute only in the three Periodic Builds, in each on Windows and Linux. This makes the daily profiling metrics load `1805x3x2 = 10,830`, likely growing to `~110,000` daily within a year.

#### Estimated Total Metrics Load

Metrics systems commonly store metrics with dimensional values, grouping a "single metric" as multiple related values and not only as individual KVPs. Under this model, daily metrics would be around `5,200,000` which is heavily dominated by test-metrics. This reduces to around `35,000` (a greater than 99% reduction) if only test modules and failures are logged, and not all individual tests.

If test metrics are allowed to naturally grow, this could reach 6,000,000 daily metrics within a year, or perhaps 150,000 if only test modules and failures are recorded. This would be around 42,000,000 vs 1,050,000 metrics per week, 180,000,000 vs 4,500,000 per month, 2,200,000,000 vs 55,000,000 per year. These metrics would exist across four or five metrics types (Pipeline Run, Job-Stage Run, Test Result, Profiling Result) with Test-Module Result being important if the reduced load is selected.

#### Estimated Metrics Cost

While a metrics solution has not yet been selected, here is one off-the-shelf estimate:

AWS CloudWatch primarily charges per type of custom metric, as well as a small amount per API call that uploads metrics data. Each post request of custom metrics is limited to 20 gzip-packed metrics, for which we should see a nearly 1:1 ability to batch data from test-XMLs. [Monthly CloudWatch cost estimates](https://calculator.aws/#/createCalculator/CloudWatch) for metrics plus a dashboard and 100 alarms are $112 for full metrics (4 types with 10MM API calls) vs $14 for failure-only (5 types with 250k API calls). This is estimated across four or five metrics types (Pipeline Run, Job-Stage Run, Test Result, Profiling Result) with Test Module Result being added if the reduced load is selected.

However it is important to limit the types of custom metrics in CloudWatch (and likely any other backend as well). While it could make dashboard partitioning and alarm-writing easier, monthly costs would be extremely high if individual metric-types were all stored separately. For instance if every unique metric were accidentally stored with a unique key (10MM types), the monthly cost could be over $240,000!

While out of scope for this RFC, this is a critical dependency which must have its usage and access limited: https://github.com/o3de/sig-testing/issues/34
