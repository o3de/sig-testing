# SIG Testing QA Charter

This charter adheres to the Roles and Organization Management specified in <sig-governance>.
 Team information may be found in the <readme.md>

## Overview of SIG

SIG Testing exists to make testing software easy, accessible, automated, and mandatory for all O3DE contributors. This charter aims to maintain the quality bar across all O3DE SIGs, and champions the O3DE development experience of both contributors and end users. SIG Testing provides tools and resources for testing, maintains test automation frameworks and test metrics, and also sets and enforces quality assurance policies.

## Goals

* Enable other SIGs to efficiently protect their own code from defects
  * Expand what can be tested
  * Identify inefficient tests and help them execute faster
  * Set policies for consistent quality standards between SIGs
  * Audit existing code for efficiency and correctness?
* Ensure automated testing can run in the Automated Review (AR) build pipeline and Periodic (Nightly) build pipelines
  * Maintain the pre-validation system and its rules
  * Define the post-build test suites executed in the AR pipeline, and suite requirements
  * Resolve testing issues that block the AR pipeline
  * Prevent tests from being improperly disabled, bypassed, or ignored
  * Define requirements of AR test nodes
* Provide test automation frameworks and tools across all supported platforms
    * Standardize and unify test automation, to limit divergent or parallel testing effort

## In Scope

* O3DE Testing Software
  * Test-specific frameworks, tools, and middleware
  * Performance benchmark frameworks, tools, and middleware
  * Metrics and reports created on the local machine
    * Test results and artifacts
    * Code coverage
    * Logfile root-cause analysis and failure summaries
  * Retrofit new software patterns onto untested features, to expose them for testing by other SIGs
  * Automated tests verifying contributors can write test automation, these tests target:
    * The testing framework itself
    * Individual test tools
    * AR pipeline functionality
    * Working example tests
* Automated Review (shared ownership with SIG-Build)
  * Pre-validation phase of AR
  * Test phase of the AR pipeline and periodic (nightly) builds
  * Metrics, reports, and dashboards for tests that execute remotely in AR
* Pull Request Reviews from other SIGs
  * SIG-Testing may be added to any pull request to ask for testing advice
* Documentation
  * Docs for each category “In Scope” above
  * Test writing guides
  * Roadmap of SIG features

## Cross-cuttong Processes

* Testing policies across all other SIGs, such as:
  * Handling changes that break tests in seemingly unrelated code
  * Resolving intermittent failures
  * Auditing existing tests
  * Maximum test duration (timeouts)
  * Minimum performance requirements
  * How to prioritize test cases
  * How and how often to organize exploratory manual testing “bug bash” exercises
* Releases
  * Define and verify quality bar of branches marked for release

## Out of Scope

* Maintain all individual tests on behalf of other SIGs
  * Approve every pull request which targets the main development branch
  * Track which tests are not automated/automatable in a feature area
* Develop non-testing extensions to features owned by other SIGs
* Maintain the entire Automated Review pipeline, including Jenkins and its hardware infrastructure fleet
* Provide headcount to execute manual test plans
* Service Level Agreement (SLA) for other SIGs to act on known issues

## SIG Links and Lists:

* Joining this SIG
* Joining Slack/Discord
* Mailing list
* Issues/PRs
* Meeting Agenda & Notes

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
* Attend at least one charter meeting a month (how often will these meetings be held, weekly?)
  * Take turns recording meeting minutes
* Set and establish team meetings and future activities
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
