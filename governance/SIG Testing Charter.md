# SIG Testing Charter

This charter adheres to the Roles and Organization Management specified in <sig-governance>. Team information may be found in the [readme.md](https://github.com/o3de/sig-testing/blob/main/README.md)

## Overview of SIG

[SIG Testing](https://github.com/orgs/o3de/teams/sig-testing) exists to make testing software easy, accessible, automated, and mandatory for all O3DE contributors. This charter aims to maintain the quality bar across all O3DE SIGs, and champions the O3DE development experience of both contributors and end users. SIG Testing provides tools and resources for testing, maintains test automation frameworks and test metrics, and also sets and enforces quality assurance policies.

## Goals

* Enable other SIGs to efficiently protect their own code from defects
  * Expand which software can be tested
  * Expand the ways in which software can be tested, such as:
    * Traditional binary testing (case pass/fail)
    * Qualatative performance testing (scenario consumes +/-X% resources)
  * Audit existing code for efficiency and correctness
    * Identify inefficient tests and help them execute faster
  * Set consistent quality standards between SIGs
    * Raise this standard over time
  * Make test metrics and reports legible at a glance
* Enable public customers and contributors to run existing tests and write new tests
  * Provide all tests, ready to run
  * Contributing tests is simple
  * Projects using O3DE are easy to test
* Ensure automated testing can execute in the Automated Review (AR) build pipeline and Periodic (Nightly) build pipelines
  * Maintain the pre-validation system and its rules
  * Define the post-build test suites executed in the AR pipeline, and suite requirements
  * Resolve testing issues that block the AR pipeline
  * Prevent tests from being improperly disabled, bypassed, or ignored
  * Define requirements of AR test nodes
* Provide test automation frameworks and tools across all supported operating systems
    * Standardize and unify test automation, to limit divergent or parallel testing effort
    * Enable OS-agnostic tests

## In Scope

* O3DE Testing Software (software which helps write tests)
  * Test-specific frameworks, tools, and middleware
  * Performance benchmark frameworks, tools, and middleware
  * Metrics and reports created on the local machine
    * Test results and artifacts
    * Code coverage
    * Logfile root-cause analysis and failure summaries
  * Retrofit new software patterns onto untested features, to expose them for testing by other SIGs
  * Automated tests which verify contributors can write test automation, such tests target:
    * The testing framework itself
    * Individual test tools
    * AR pipeline functionality
    * Working example tests
* Automated Review (shared ownership with SIG-Build)
  * Pre-validation "Early Warning System" phase of AR
  * Test phase of the AR pipeline and periodic (nightly) builds
  * Metrics, reports, and dashboards for tests that execute remotely in AR and Nightly builds
* Pull Request reviews from other SIGs
  * SIG-Testing may be added to any pull request to request testing advice
* Documentation
  * Docs for each category “In Scope” above
  * How to reach out to this SIG
  * Test writing guides
    * Test Plans for a component
    * Individual tests for a feature
    * Covering multiple operating systems
  * Roadmap of SIG features

## Cross-cutting Processes

The following responsibilities are driven by SIG-Testing across all other O3DE SIGs

* Issue tracking guidelines
  * How to use Templates and Labels
* Test policy recommendations, such as:
  * Handling changes that break tests in seemingly unrelated code
  * Resolving intermittent failures
  * Auditing existing tests
  * Maximum test duration (timeouts)
  * How to prioritize test cases
  * How and how often to organize exploratory manual testing “bug bash” exercises
* Releases
  * Define and verify the quality bar of branches marked for release

## Out of Scope

* Provide headcount to execute manual test plans
* Maintain all individual tests on behalf of other SIGs
  * Write new automated tests for each feature
  * Fix individual broken tests or the product bugs they point to
  * Approve every pull request which targets the main development branch
  * Run additional automated tests not configured to execute in Jenkins
  * Track which tests are not automated/automatable in a feature area
  * Monitoring individual test health or their metrics
    * Defining software performance requirements
* Develop non-testing extensions to features owned by other SIGs
* Maintain the entire Automated Review pipeline, including Jenkins and its hardware infrastructure fleet
* Defining or enforcing a Service Level Agreement (SLA) for how fast other SIGs to act on their own issues
* Policing other SIG's goals or priorities

## SIG Links and Lists:

* [Joining this SIG](https://github.com/orgs/o3de/teams/sig-testing/members)
* Joining Slack/Discord
* Mailing list
* [Issues](https://github.com/o3de/sig-testing/issues) and [Pull Requests](https://github.com/o3de/sig-testing/pulls)
* [Meeting Agendas & Notes](https://github.com/o3de/sig-testing/labels/mtg-agenda)

## Roles and Organization Management

SIG Testing adheres to the standards for roles and organization management as specified by <sig-governance>. This SIG opts in to updates and modifications to <sig-governance>

### Individual Contributors

Additional information not found in the sig-governance related to contributors:

* Include sig-testing on all pull requests modifying O3DE testing software, listed above as “In Scope”
  * Concisely describe the proposed contribution
  * Include new unit tests and testable examples alongside new testing features
* Optionally include sig-testing on pull requests for other features which need testing feedback
  * Clarify any questions being asked, or highlight tests that need feedback
* Log issues to track bugs and feature requests
* Provide feedback on open issues

### Maintainers

Additional information not found in the sig-governance related to maintainers of this SIG:

* All responsibilities of Individual Contributors, plus those of Maintainers listed below
* Review incoming pull requests from contributors, a minimum of twice per month
* Actively contribute to at least one activity defined above in the “In Scope” section
  * Submit at least one pull request per month modifying code, tests, policies, or documentation
* Attend at least one charter meeting per year (how often will these meetings be held, monthly?)
  * Take turns recording meeting minutes
* Organize SIG-Testing meetings in Discord
  * Meet with other SIGs when appropriate
* Resolve disputes among contributors and maintainers
  * Bring deadlocked disputes to the SIG Chair for review

## SIG Chairs

Additional information not found in the sig-governance related to SIG Chairs:

* All responsibilities of Individual Contributors and Maintainers, plus those of the Chair listed below
* Work with sig-governance to set and maintain future direction
  * Report back any changes to the group
* Set all agenda related to roadmap
* Be a tie breaker when conflicts within the group deadlock
* Disputes with the chair can be brought to the sig-governance board for review

## Subproject Creation

Placeholder for additional information not found in the sig-governance related to subproject creation.  There are currently none.

## Deviations from sig-governance

Placeholder for explicit deviations from sig-governance guidance. There are currently none.
