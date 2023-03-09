# RFC: Public Metrics

## Summary

Contributors to O3DE have no simple dashboard summarizing the health of its Jenkins Continuous Integration pipeline. In open source projects, such health metrics are commonly summarized into [workflow status badges](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge) in a [project's readme](https://github.com/buildbot/buildbot_travis#readme). The majority of currently available info is tied to the short (~15 day) [Jenkins' job history](https://jenkins.build.o3de.org/job/O3DE/job/development/) spread across disparate Jenkins jobs, which do not provide a simple overall summary. These Jenkins-hosted pages are also fragile to issues related to Jenkins availability, including server restarts. There is an existing long-term plan to onboard to Linux Foundation's LFX Insights metrics tool if its maintainers deliver an adapter for custom metrics data. However waiting to onboard is in tension with an existing need for actionable health metrics.  Delivering contributor-facing metrics unblocks improving the health of O3DE and its tests.

### Customer cost of waiting

Non-contributing customers who seek to use O3DE not likely interested in these metrics, and would primarily feel pain from a lack of stability in the code. They may be interested in seeing a workflow status badge summarize the build health of a branch, which could help them discover recent known issues. Non-contributing customers who are personally modifying the engine may be interested in hosting up their own version of these metrics for their own pipeline, which could speed up their own DevOps workflows and reduce their costs. These are assumptions, and not backed up by user studies nor interviews.

However there are secondary effects felt by downstream customers, which can be improved through metrics. The most important is from contributors efficiently improving the product. If contributors can more efficiently build a better product, then non-contributors will have a better experience. Additionally, publicly posting O3DE's metrics is an opportunity to increase trust in the stability of the product. And even when the software becomes unstable, posting status indicators and metrics can reduce negative surprises which rapidly erode trust. Changes in stability could also be used to highlight areas to contribute.

Overall Customer Pain: Low

### Contributor cost of waiting

Publicly-viewable pipeline metrics enable O3DE contributors, especially SIG Maintainers, to focus effort on areas of low stability and low performance. Without metrics reports which are trivially queryable, it is common for distributed collaborators to each refer to different sets of locally-collected data. This increases the friction of communication and planning, and ultimately impacts engineering velocity of delivering the correct product. Delaying metrics availability delays improving the speed and accuracy that contributors improve O3DE.

Metrics can also serve as a type of scoreboard, which can gamify the act of improving the code. The sooner a metrics service is in place, the more quickly O3DE can start prompting contributors with measurements of where to improve.

Overall Contributor Pain: Medium

## Feature Design

While there may be future need for other types of public metrics, this proposal focuses only on pipeline-generated health metrics including:

1. Top-level Jenkins Job Status, per long-lived Branch (development, stabilization)
2. Inner-results of each Job (build pipeline) and each separate Step taken within those jobs
3. Individual results from tests and performance benchmarks.

Historical data is fundamental to analyzing performance results, and also is important for [identifying recurring failures](intermittent_failures.md) in Jenkins infrastructure and individual tests. To provide this, we should provide all raw metrics from the previous couple months. Condensed metrics reports should also be produced, which can persist for the lifetime of the project to summarize historical data. Due to the time-limited nature of health metrics, and to save on costs, old raw data can be discarded.

## Technical Design

This proposal only covers metrics which are harvested from the Jenkins build pipeline maintained by O3DE. It explicitly excludes other metrics, such as:

* **No** metrics collected from public users of the O3DE Engine and Tools
* **No** metrics collected from applications created via O3DE which release to public consumers

No part of the O3DE applications should "phone home" – only the Continuous Integration (CI) pipeline should transmit these health metrics. O3DE applications, tools, and tests come with options to save local data about their health, and a CI pipeline can persist this data. These metrics are intended for use by public contributors of the O3DE project, particularly the maintainers of Special Interest Groups (SIGs). No data should be stored which identifies individual users, and no confidential information should be stored.

These metrics should only be recorded from CI pipeline infrastructure maintained by the Linux Foundation / Open 3D Foundation. Public metrics should not be recorded from privately owned hardware. This includes any discrete physical (non-cloud-based) machines used for performance measurements.

This design intends to comply with the [Linux Foundation policy for telemetry data](https://linuxfoundation.org/telemetry-data-policy/). To ensure it continues to do so in the future, the summary above should also be documented alongside the public code for any metrics scraping or uploading tools.

### Pipeline Metrics Overview

![Data Flow Overview](overview.png?raw=true)

The image above is a high-level data flow overview of how O3DE Maintainers can generate pipeline metrics. Maintainers primarily generate metrics by requesting builds of O3DE build pipeline against proposed changes through Automated Review (AR) CI pipeline. An identical fully-automated Jenkins job also regularly runs against the branch after changes get merged, generating health metrics for the actual state of long-lived branches such as Development and Stabilization. Additional fully-automated Jenkins jobs will periodically run a broader set of CI pipelines on a slower cadence, providing metrics from variant builds and lower-priority tests in the Periodic test suite.

Requesting builds is notably accessible only by O3DE Maintainers, and not all public contributors. The metrics backend and its alarms, which send a notification by e-mail or messaging app, should similarly not be accessible by public contributors. Only O3DE SIG Maintainers should have direct access to modify the metrics backend. Any O3DE Metrics Dashboard should be publicly viewable, and may be as simple as a few status badge indicators in a project's `readme.md` to summarize the state of health alarms.

#### Collecting Metrics from a Jenkins Job

![Data Flow Overview](jobdetail.png?raw=true)

The image above shows a data flow zoomed in from the previous overview, focusing on how metrics will report from inside a stage of a pipeline in a Jenkins job.

It is unclear whether it is more effective to incrementally upload metrics from each individual Jenkins stage, or with a dedicated "Metrics Stage" that runs at the end of a pipeline. However this is primarily a question of how metrics code is maintained, and how reliably they upload. Regardless of approach, the security footprint should be nearly identical: Jenkins has access to the relevant auth tokens through a Secrets Manager plugin, which it provides for the Metrics Uploader.
The Metrics Uploader tool is specific to the O3DE Jenkins CI system and its metrics back-end, so it may be appropriate for it to exist in a separate repo. If it lives in the O3DE repo, it should exist near the Jenkins-specific scripts in /o3de/scripts/build/Jenkins.

#### Modifying Metrics Values

![Data Flow Overview](useraccess.png?raw=true)

Contributors are capable of suggesting significant modifications to pipeline metrics through pull requests. However all changes are required to be approved by SIG Maintainers, with assistance from the Automated Review build pipeline.

Most any merged change can modify the performance or stability of the code, which in turn modifies the recorded values of existing timing and pass/fail metrics. These code changes can also result in adding new metrics keys from new tests and benchmarks. As the Jenkinsfiles that configure the Jenkins instance are stored in GitHub, this also means pull requests can modify which stages of the pipeline execute, which can change which metrics are logged or disable metrics from being logged.

Access to the code which defines the metrics to upload notably allows changing the set of metrics keys which are uploaded to the metrics backend, as well as the total volume of metrics which are produced. Both can result in increased costs to store metrics data. Size limits should be put in place to prevent accidentally recording an unreasonable amount of metrics.

The public metrics dashboard (even if it is just a status badge) can also be modified through merging a pull request. As everything can be modified by pull request, it will fall to SIG Maintainers to stay vigilant about the changes they merge.

#### Reducing Metrics Volume

As described below in the appendix, the vast majority of metrics data would come from passing tests. Storing only non-passing results creates a ~99% reduction in data, for a minor reduction in data fidelity. The tradeoff would be less ability to detect when tests get renamed, added, removed, or never-fail. However reports can still reconstruct pass-rate metrics for individual failing tests.

### Proposed Metrics Backend: Metrics CSVs saved to S3

The cheapest and most portable option appears to be directly saving metrics in CSV format to S3. Highly portable due to most every off-the-shelf tool being able to ingest CSV data, and files are easy to move between different data warehouses. The bucket can set to be readable by anyone. To reduce costs further, the bucket can enable the [Requester Pays](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RequesterPaysBuckets.html) option, which will reduce the risk of abuse from anonymous access.

This solution is affordable primarily by not charging for extra features, including unused features. This route would require adding another solution to report on these metrics to a dashboard. One affordable path is to have our own automated script periodically ingest metrics and produce the status report, which then gets hosted in S3 as a static HTML page. To limit impact on the pipeline, this script should run on AWS Lambda (or GitHub Actions, which may be cheaper for O3DE). Zipping CSV files before upload enables trading costs between storage size and additional time in pipeline + lambda.

Estimated monthly cost: $2/month for pipeline, benchmarks, sparse test metrics stored in S3, all processed by Lambdas which periodically set alarm-flags and store a report in S3. ($50/mo for full test metrics).

Adding [S3 Transfer Acceleration](https://aws.amazon.com/s3/transfer-acceleration/) would increase cost by around $80/mo ($3320/mo for full test metrics). This is not recommended unless it also enables significant savings elsewhere.

Pros

* Immediately ready to use via existing O3DE AWS account
* Simple to implement, simple to set a data retention policy
* Low cost
* Bulk file uploads are relatively fast (common use case) and can be sped up
* Easily adapts to other metrics back-ends and dashboards

Cons

* Need to publish a custom report, or find a dashboard tool
* Need to configure custom alarm scripts, or find an alarm tool
* Must define our own storage pattern for files

## Are there any alternatives to this feature?

### Other Backend Options

#### Wait for LFX Insights features

Linux Foundation has its own metrics dashboard, which is not currently ready for handling custom metrics. This [feature has been requested](https://portal.productboard.com/bqt1cgergoszsqheudb7leux/c/118-bringyourownconnector-byoc), but there is no commitment to deliver it.

Pros

* Linux Foundation has requested this service be used, and will be more likely to approve any ongoing metrics expenditures related to it.
  * Should be compliant with the Linux Foundation's own data policy
* No extra cost incurred directly by O3DE
* Not owned or maintained by Amazon or another private interest
* Dashboard is publicly accessible by default
  * Has a decent [dashboard UI](https://insights.lfx.linuxfoundation.org/projects/o3de-f/dashboard;quicktime=time_filter_3Y)

Cons

* Waiting on unscheduled feature request from LFX, with no established ETA
* Unproven use case of custom pipeline/test/perf metrics and alarms
  * Likely that certain use cases will be blocked, and need further iteration by LFX team
* Customers and contributors would wait until at least 2024 for metrics features to come online
* Downstream customers may not be able to use the same dashboards by copying configuration files (LFX appears only for Linux Foundation projects)

#### AWS CloudWatch Metrics

Cloudwatch provides a simple metrics solution, which includes the ability to set alarms. A "single metric" consists of a key, value, name, namespace, and up to ten "dimensions" which are a set of secondary/metadata KVP. A moderately-accurate UTC timestamp is effectively added for free. AWS CloudWatch primarily charges per **type** of custom metric, as well as a tiny amount per API call that uploads metrics data and for dashboards and alarms. Each post request adding custom metrics is limited to 20 gzip-packed metrics. Monthly CloudWatch costs would estimate metrics plus a dashboard and 100 alarms at $112 for full metrics (4 types with 10mm API calls) vs $14 for failure-only (5 types with 250k API calls). This is estimated across four or five metrics types (Pipeline Run, Job-Stage Run, Test Result, Profiling Result) with Test Module Result being added if the reduced load is selected. The less-homogenous data in the performance metrics may prompt splitting them out into separate metrics categories. Increasing the types of metrics from 5 to 1810 would increase the cost by around $540 per month and is not advised. Optimizing this may require additional investigation.

It is ***very*** important to limit the types of custom metrics in CloudWatch. While new metrics groups could make dashboard partitioning and alarm-writing easier, if individual metric-types were all stored with a unique key then monthly costs could increase by $7000.

Estimated monthly cost: $20/month for pipeline, benchmarks, and sparse test metrics ($120/month for full test metrics)

Pros

* Immediately ready to use via existing O3DE AWS account
* Proven tool with extensive documentation
* Affordable if used correctly
* Can share everything except access keys, such that customers can set up their own dashboards for their own projects

Cons

* Requires careful use to avoid generating a significant increase in metrics-spend for O3DE / LF
  * All SIG Maintainers have the ability to merge changes to metrics (risk can be partially mitigated by structured upload scripts)
* Short-term data storage is enforced to 14 days, after which historical metrics get aggregated into 5-minute groups for 63 days, after which they are aggregated into 1-hour groups for up to 15 months.
  * Additional storage costs to warehouse individual metrics or reports longer-term
* A public dashboard requires [configuring access control](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-dashboard-sharing.html) to AWS accounts, which seems inappropriate.
  * Could instead periodically publish metrics data through a Lambda-hosted script into a static HTML page hosted in S3, for a small extra cost
* Difficult to create custom widgets and views (prefers only simple graphs and alarms)
* May have data availability/latency concerns (metrics tied to single region though us-east-1 should be fast enough for most users)

#### AWS OpenSearch

AWS OpenSearch (often called "ElasticSearch ELK" or "Kibana") is a service that runs live on EC2 instances and surfaces a web dashboard. It is a live toolbench-style service favored by data scientists.

Estimated monthly cost: Upward of $1000/month, dominated by EC2 instance costs for two on-demand (non-dedicated) instances for a production and staging environment.

Pros

* Immediately ready to use via existing O3DE AWS account
* We can share everything except access keys, such that customers can set up their own dashboards

Cons

* High cost to host the dashboard
* Linux Foundation has previously declined a proposal for AWS OpenSearch dashboard, and would be unlikely to approve use of a similar service

#### AWS Athena

AWS Athena provides a big data framework for storing and parsing values in S3, but with no built-in dashboard tools. It is not a good option.

Athena has a monthly cost of $5 per terabyte scanned, plus S3 storage at $4-25 per terabyte varying on access speed. However it will heavily depend on what additional custom logic is written for dashboards and lambda-alarms to interact with its data, which would also incur a cost.

Estimated monthly cost: $50/month, with significant linear cost growth month-by-month unless we manually prune or merge our data. ($1800/month for full test metrics, similarly linear)

Pros

* Immediately ready to use via existing O3DE AWS account
* Proven tool with extensive documentation
* More control over what you build
* We can share everything except access keys, such that customers can set up their own dashboards

Cons

* Significant engineering cost to setup and maintain over time
* Need to define an S3 data schema to limit costs
* Need to create custom dashbaords
* Need to write Alarms as queries for a Lambda to run

## Appendix: Estimated metrics load from Jenkins pipelines

This section estimates how much metrics data would be logged across multiple pipelines, which each have many jobs and many stages.

### Jenkins Jobs Metrics

Pipeline run metadata should be stored per pipeline run, which should include at minimum:

* Jenkins Pipeline execution ID (URL?)
* Branch Name (Development/Stabilization/etc.)
* Git commit hash
* UTC Start Time
* Total Duration
* Result (Success/Fail/Abort)
* Build parameters, such as:
  * CLEAN_OUTPUT_DIRECTORY
  * CLEAN_ASSETS
  * CLEAN_WORKSPACE
  * RECREATE_VOLUME

Each pipeline currently executes multiple build-jobs in parallel. The following information is useful for each stage from each job:

* Pipeline Stage Name (including variant info such as job name "Windows_Profile", and branch name "Development")
* UTC Start Time
* Duration
* Result (Success/Fail/Abort)
* Pipeline run metadata ID (foreign key)

#### Pipeline Metrics Volume

There are multiple pipelines in Jenkins, which currently have approximately following daily cadences:

* 40x Automated PR Review
* 20x Branch Indexing
* 1x Nightly Clean Development
* 1x Nightly Incremental Development
* 1x Nightly Clean Stabilization

There are currently a total of 29 stages across the seven parallel jobs in Automated Review (Pull Requests) and Branch Indexing (Merge Consistency Checks) runs. There are approximately 40 pull request runs and 20 branch updates per day (across two branches).

Nightly builds have many more jobs, currently around 180 stages across 46 parallel jobs. Of these Mac jobs comprise 29 stages across 9 jobs, which may not be used in the public infrastructure. It is difficult to estimate how this set of stages will change over time.

The daily load factor should be around `60x29 + 180x3 = 2280`. If Mac continues to not be included, this would be around 2200 sets of Jenkins-level metrics per day.

### Test Results

Results from individual tests will constitute the bulk of the metrics data. Currently only two parallel jobs execute tests in Automated Review, but each job contains thousands of tests. Transferring and storing this sizeable amount of data has time and storage costs, and likely needs its own optimizations.

A decent amount of the information from test XMLs are relevant to metrics, however the file data also contains significant XML boilerplate. Instead of directly saving XML files, individual metrics can be extracted. As transforming this data has a time and hardware cost, it may be appropriate to execute in a Lambda (or GitHub Action) executed after the build pipeline.

The following information is desired from every test:

* Result (Pass/Fail/Error/Skip)
* Test Name
* CTest Module Name
* Duration
* Pipeline Stage Name (foreign key)
* Pipeline run metadata ID (foreign key)
* Public SIG owner

#### Test Metrics Volume

There are currently around 43000 tests that run in each Automated Review and Branch Update test-job, which is expected to slowly grow over time. One path to reducing the scope of test metrics is to bundle these metrics into only reporting on the module that contains sets of tests, for which there are currently 135 modules in Automated Review. This is a major tradeoff of data quality for size, and would reduce the ability to track which specific tests are failing. To reduce data loss, it should also be paired with saving the total number of pass, fail, error, and skipped tests.

A less extreme way to reduce the volume of data is to only store non-pass results, and assume that a "missing key" is a passed test. This reduces data fidelity in the cases where tests are being renamed, added, removed, or never-fail, but retains the ability to calculate pass-rate metrics for individual failing tests. This would result in a variable load of metrics which increases as more tests fail per run. Currently around 1/10 of test runs encounter a test failure. When such failures occur, the current average number of failures is around 2.5. It is quite often 1, and is rarely above 100. This should result in around a 99% reduction in data.

Another potential option to reduce data volume is only collecting test metrics from branch-update and periodic jobs, and not include the failures found during pull requests. This has the benefit of excluding the artificially noisy failures that occur in not-yet-ready pull requests which frequently fail for legitimate reasons. However the drawback would be not having data to highlight tests where developers get tripped up in pull requests. This would result in around a 50% reduction in data.

Nightly jobs currently contain an additional nearly 2000 tests, across 34 modules. While these are broken out across a few parallel jobs to increase speed, the tests run across four variants (linux_profile, linux_profile_gcc_nounity, windows_profile, windows_debug)

The amount of test results should slowly expand over time, as new modules are added. While applications and tests can be made more efficient and reliable, this also tends to encourage adding more features and tests.

The daily load factor for saving all test-metrics would be 43000x2x60 + 2000x4x3 = 5,184,000 test-level metrics per day.
If only modules are reported, this would be 135x2x60 + 34x4x3 = 16,608 module-level metrics per day.
If modules and only test-failures are reported, this would be 0.1x2.5x60 + 135x2x60 + 34x4x3 ~= 16,625 module-plus-failure metrics per day. At a 99.7% reduction and a minor tradeoff, this approach is recommended.

Performance Profiling Results

There are multiple performance benchmarks, each of which emits at least one metric describing its execution. Currently there are:

* 1795 Micro-Benchmark metrics (across 10 modules)
  * Duration to complete OR iterations per unit of time OR bytes per unit of time
  * Maximum memory footprint
  * SIG Owner
* 10 Workflow benchmark metrics
  * Duration to complete
  * Maximum memory footprint
  * SIG Owner
* Each of these metrics may additionally want to record:
  * An error-signal for when exercising the benchmark crashes, hangs, or was fails to run

These benchmarks execute only in the three periodic (nightly) builds as often as twice per day, inside each running on Windows and Linux. This makes the daily metrics load around 1805x3x2x2 = 21,660.

There may be additional per-module metrics to record similar to test metrics. However there is no value in aggregating this data into per-module metrics, as the primary value for benchmarks is from tracking the individual values over time.

Providing the ability to upload and track metrics will encourage the current number of performance metrics to grow, as such metrics otherwise have low utility. A pessimistically-high estimate is that this will expand by 10x within a year.

Estimated Total

Metrics systems commonly store metrics with dimensional values, grouping a "single metric" as multiple related values instead of only individual key-value pairs. Under this model, daily metrics would be around 5,200,000 and heavily dominated by test-metrics. If only test modules and failures are logged, and not all data for individual tests, this would instead be around 30,000. If metrics are allowed to naturally grow, within a year this would likely reach 6,000,000 daily metrics, versus 50,000 if only test modules and failures are recorded.

This pessimistically-high estimation equates to around 42,000,000 vs 350,000 metrics per week (180,000,000 vs 1,500,000 per month which is 2.2bn vs 0.02bn per year). Estimated (compressed) data volume per month is 83 TB for full test metrics, and 2 TB for sparse test metrics. Given the volume of raw metrics and the time-limited value they provide, they should not be stored indefinitely. It would be appropriate to only persist condensed reports long-term.
