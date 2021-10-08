# SIG-Testing meeting notes from 2021-09-24 @ 1830 UTC

Moderator: amzn-dk
Note Taker: amzn-dk

## Previous Meeting
In our last meeting we discussed AR Nightly reporting and received Automated Testing updates from the Amazon SDET team. Discussion centered around how to better educate contributors on how to create tests and identify coverage.

## AR Escaped Bug Coverage
We discussed the need to begin reporting on bugs which are not caught by the Main and Periodic suites in AR as these represent a gap in coverage.

* We should get more documentation around creating unit tests so that we can give the engineers the tools and how to do it. 
* We currently have no public facing documentation or training, last time we did a training tour was 1-2 years ago. We should revisit this publicly to help educate people on how to use the systems and which ones we're using, with the intent being that we have it available and can remove the excuse of not knowing how to do it or what to test.

## Code Coverage Standards
Next we discussed the concept of setting code coverage targets for new PRs being submit into the project.
* Industry standard is 80%
* Focus on targets and enabling SIGs to evaluate their coverage gaps, not a hard requirement
* Focus the code coverage to the diff/PR you created not all pre-existing / legacy code as there it is known we have tech debt in this area and it is unrealistic to tackle it all in one effort
* Recently discovered Atom unit tests were not hooked up to CMake - we need to ensure that the tests not only exist but that they are hooked up and being run
* Look into getting metrics of tests/test names being run with a diff week over week (new, removed, updated) to ensure tests don't get disabled without visibility
* We can give guidance on this to SIGs with current resources, but we may not want to give a central solution as applying code coverage tooling into our AR runs comes at a performance cost

## Misc updates
RFC from Atom team for Unit Tests was reviewed through the Graphics and Audio SIG
* Pushback from the meeting and review that we need to justify with metrics why unit testing would be useful for the area
* Active discussion going on about which mechanisms need to be built to gather code coverage locally, do we need to expose that in AR etc.
* Will be reviewed again in next G&A SIG meeting

Nightly(Periodic) AR run has not been onboarded to the O3DE pipeline due to costs and is still being run by Amazon. Still investigating reducing costs for the future. We still have on-call and need to manually enter these issues.
* Continued to press the need for automated entry of issues in GHI for Periodic suite failures
* There is tool for auto-creation of bugs for build breaks against the branch update runs, we may be able to extend this to work with nightly failures
* Initial implementation would cut a simple ticket without much information and the ability to prevent dupes
* There's a plan to put sig ownership in GH code owners files that would include all files and tests. Not implemented yet but would be the path to enable us to assign owners to test failures.

## Wrap up
Overall we re-emphasized the need for automated entry of nightly failures as the on-call comes at a high manual support cost.

Currently plan regular SIG-Testing meetings every two weeks at this time, though impromptu meetings are encouraged.
