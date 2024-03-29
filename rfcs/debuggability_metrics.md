# Metrics for Debuggability

## Summary

Currently, most design decisions and work for debuggability are done through either grass-roots movements where small groups of developers make a change, or a contributor pushes for a specific improvement. It is consequently unclear what metrics have been important in past decisions. We want to design and implement metrics so that such decisions can be supported by appropriate data and to measure the effectiveness of changes made because of these decisions.

Selecting and measuring appropriate metrics for debuggability is challenging. Ultimately, debuggability has a significant human element that is difficult to quantify. However, it is still valuable in measuring what can be measured, as well as acknowledging the incomplete picture that these metrics provide.

There is also a wide range of improvements that can be done for debuggability timelines that cross over into other areas. For example, faster build times also improves debuggability timelines. However, to limit the scope of this RFC, we are going to limit this document to discussing elements that are unique to debuggability.

We are suggesting to measure a few key debugging times and polling customers on their debugging experience.

## What is the motivation for this suggestion?

The primary is to enable metrics-driven decisions around improvements to debuggability. Diving a little deeper, these metrics will allow us to more intelligently focus our efforts in improving debuggability, specifically in improving the timelines for discovering and fixing bugs. It will also allow us to prioritize which problems are most severe.

We also want to be able to track the impact of decisions we make to improve debuggability. Measuring metrics and their subsequent improvement is one way to do this. This will allow us to provide visibility to the larger organization by a metrics-driven approach and provide support for claims of debuggability improvements.

## Design Description

### Debugging Steps

Here are some common steps of finding and fixing a bug:

1. The error is reported. This is done by Jenkins sending out an e-mail, although developers can also view the Jenkins build through its web UI.
2. The log is investigated. This can be found through the web UI for Jenkins. (or in a GitHub issue)
   * This may include artifact investigation, which is also found through the web UI for Jenkins
3. The bug is reproduced locally. This step is not always required, but often is. Depending on the developer's workstation state, the nature of the bug, and the developer's preferences, the following sub-steps may be required:
   1. Sync the commit where the bug was found.
   2. Build the commit in the appropriate flavor.
   3. Add breakpoints. This is a less common step.
   4. Add new output. This is a less common step.
   5. Run the appropriate executable(s).
   6. Run the appropriate test(s).
4. Make changes to actually fix the bug.
   * If no test existed to catch the bug, also add an appropriate test(s) to prevent the bug from regressing.

### Measuring Debugging Time

Each of the steps above can be measured in time, although the value of those data points will vary wildly. Many of them are too dependent on the bug, can be done in parallel, or are out of scope for debuggability.

Local reproduction is a borderline case - things like build time are out of scope, but making sure that the bug can be reproduced locally reliably are in scope. Further, there's a ton of variables involved that will make any metrics we do measure exceptionally noisy. Did they have a partial sync already? Did they have a build already? Do they prefer to debug via logs or debug via the debugger? All of these will cause the time to vary wildly while not actually indicating the ease of the debugging as defined as in-scope for this RFC. Given that local reproduction is more of a binary question (Can we reproduce the bug locally?) and given the scope of this RFC, we will not be including it as a suggested timed metric.

### Debugging Time Metrics

Thus, this leaves us with 3 areas to measure for time:

1. Error reporting time: This step would have three parts. Currently, Jenkins only outputs an error e-mail when all builds in a run are finished, but changes could be made to report an issue as soon as a build in a run has failed.
   1. Time when failure is first signaled inside Jenkins job
   2. Time when Jenkins sends an e-mail reporting the failure to the user (or a bot posts a failure message)
   3. Time when user takes action on the notification and starts working on their machine (modifying code, or starting a build)
2. Log-finding time: This step would have two parts. We expect this to be a small portion of the total time consumed, but it is still valuable to reduce it where possible.
   1. Time it takes from notification to finding the correct log
   2. Time it takes (after finding the correct log) to parse enough information in the log to take action locally
3. Artifact-finding time: We expect this to be a small portion of the total time consumed, but it is still valuable to reduce it where possible.

The first is a binary option - we can either send the e-mail at the end of the build, or upon the first failure. But measuring that difference could allow us to decide if the earlier messaging time is worthwhile. The second two options are going to have to be user-reported metrics.

Since we are already including user-reported metrics, we would also include a question about whether or not the logs, once found, were useful in debugging the issue.

Metric | Description | Measured By
-- | -- | --
Failure first signaled | Time to see first failure in Jenkins | Automated script
Failure e-mail sent | How long it takes for an e-mail/message to go out after a failure is found | Automated script
Log-finding | How long it takes & how many clicks to find the appropriate log from a failed Jenkins run main page | Polling, customer reports
Log-parsing | How long it takes to find the actual error message in the appropriate log | Polling, customer reports
Artifact-finding | How long it takes & how many clicks to find artifacts from a failed Jenkins run main page | Polling, customer reports

## Steps to Achieve Goal

Even if a given objective is swift to complete, having too many steps (often in tedious clicks), can be a frustrating developer experience. Thus, we will measure the amount of steps a given goal took prior to changes and after changes. This metric has the advantage of not relying on polling, either, as we can independently measure this.

## Polling

While potentially less reliable than absolute metrics such as measuring time, polling customers for their preferences and pain points is still valuable. We would reach out to the sig-ux in order to develop questions. Measuring user preferences is not a primary skill set of our team and we would want to lean on experts to ensure an effective solution for our customers.

As part of polling, we would include instruction on how to effectively cut issues for inefficient workflows. While this would not produce immediate results, teaching our customers how to effectively provide feedback long-term will allow us to see their pain points well after the survey(s) have been completed.

### Checks

As a part of polling, we want to ensure that certain key checks are hit. These are:

* Were the logs valuable in helping you debug/reproduce the bug?
* Were you able to reproduce the bug locally, the first time?
* Were you able to reproduce the bug locally?
* Were you able to reproduce the bug?

These are, in order, valuable benchmarks to hit. Reproduction of a bug is key in fixing it. Being able to reproduce it locally allows a developer to glean important information in fixing it. Finally, being able to reproduce it the first time is helpful in simply saving developer time by not forcing them to repeat steps.

### Implementation

* Write script that scrapes Jenkins build logs to measure Failure first signaled and Failure e-mail sent metrics
* Perform external developer polling (this would be where most questions would be) (this would include log-finding, log-parsing, artifact-finding metrics)

### Possible Outcomes / Scenarios

Between metrics and polling, we will be able to generate a better picture of our pain sources and problems than we have now. While we won't be able to predict every possible outcome of these metrics and polling, we will attempt to identify several likely ones and comment briefly on our plans for those outcomes.

Outcome/Scenario | Planned Steps
-- | --
Problems are not being logged due to unclear issue-filing process | Provide/improve documentation on issue-filing, run trainings on issue-filing
Problems are not being logged due to effort required | Provide quick-links and other shortcuts, create issue templates
Problems are being logged, but are not being actioned on | Use metrics to provide data to prioritize issues higher, reach out to appropriate owners

## What are the advantages of the suggestion?

* Polls customers directly to get their experiences
* Focuses on the key aspects of debuggability and provides a limited scope
* Low investment cost

## What are the disadvantages of the suggestion?

* Not all aspects are measured with metrics, meaning that some pain points may be under-examined
* Metrics are only a partial picture of the problem, potentially supporting some solutions more than they merit (this can be mitigated by seeking customer feedback)
* Customer opinions may change over time, requiring multiple polls (may require setting a particular cadence in order to maximize responses)
* Some issues found by metrics/polling may not be relevant to the scope

## Are there any alternatives to this suggestion?

This suggestion has multiple components that are not coupled, so each component can be considered individually. This runs the risk of giving a partial picture, but perhaps the lesser amount of investment is worth the lesser result.

We could implement metrics on every step of the debugging process. This has a high risk of noise, as certain steps running long are way less impactful than others (builds can be run overnight, for example, while finding the logs and artifacts is an active step). Further, many of these metrics can be addressed by tasks out of scope for this RFC.

We could skip the detailed customer experience poll. Broad polling often has low engagement, however, so a more direct interaction is expected to provide better results, even if it is at a higher cost.

## What is the strategy for adoption?

First, this would require reaching out to coordinate with the sig-ui-ux to develop polling questions. Metrics and polling both require customer input, so this would require a customer outreach campaign. We would also have a small number of direct customer interactions. We would have to collate these results, then make our changes, then poll again to see if our changes have had the desired effects
