# Code Coverage Reporting for InstructLab Repos

## Problem Statement

We currently capture code coverage reports for many of our repos with `pytest`, but that information is not directly relayed to pull request authors. To view code coverage reports, the pull request authors must sift through the CI logs to identify code coverage. Even then, the code coverage reporting only tells the contributor the overall code coverage -- not the coverage on their branch.

## Target Goals

By implementing code coverage reporting:

* Contributors should be able to _easily_ identify which new lines of code in their branch need more coverage.
  * This means contributors should not need to dig through CI logging to determine code coverage.
  * Ideally, a bot should leave a comment to summarize the code coverage for them OR the contributor should be able to find this information without having to search through text (e.g., logging).
* Contributors should understand how well their branch code is covered relative to the current `main` branch code.

## High-Level Functional Requirements

* Code coverage should be reported in PR builds as a GitHub _bot_ comment
* New code:
  * Should show % coverage on new lines of code.
  * Should show which new lines of code are uncovered (if any).
* Existing code:
  * Should show existing % coverage on all lines of code.
* New code + existing code:
  * Should show projected % coverage if the PR is to be merged (e.g., “overall code coverage 53% -> 62% after merge”).
* The GitHub coverage reporting bot should be able to interpret at least one of these pytest coverage reporting formats:
  * HTML
  * XML
  * JSON
  * lcov
  * annotate
* The README should display a code coverage badge

## Code Coverage Reporting Bot: Options

### [Codecov](https://github.com/marketplace/codecov)

* [Example Codecov job](https://github.com/ansible/vscode-ansible/runs/33887209843)
* Pros:
  * Free to use in open source projects.
  * Codecov reports all the data on GitHub in a comment. [Example](https://docs.codecov.com/docs/pull-request-comments).
* Cons / Callouts:
  * Not applicable at this time, but if we want to use it in >1 private repo, we must purchase a customer plan.
  * According to Codecov: "All subscribers of a Pull Request get an email when Codecov posts a comment (only when posting a new comment), and there's nothing we can do about it.” (Read more [here](https://docs.codecov.com/docs/pull-request-comments#filtering-emails).)

### [Coveralls](https://github.com/marketplace/actions/coveralls-github-action)

* [Example Coveralls job](https://github.com/gophercloud/gophercloud/actions/runs/12117814428/job/33858835625?pr=3244)
* Pros:
  * Free to use via [MIT license here](https://github.com/coverallsapp/github-action/blob/main/LICENSE.md).
  * Appears to be free for open source repos, but like with Codecov, we should validate that the ToS doesn’t have fine print, etc.
  * Can report all the data in a GitHub comment. [Example](https://github.com/coverallsapp/coveralls-node-demo/pull/19#issuecomment-1035344985).
* Cons / Callouts:
  * Code coverage visuals, etc. are all viewed through the Coveralls front end (coveralls.io), so if that service were to go down, then we’re technically at their mercy.

### [SonarQube](https://github.com/marketplace/actions/official-sonarqube-scan)

(**Note**: Most expensive $$$)

* [Example SonarQube job](https://github.com/ansible/vscode-ansible/runs/34526218267)
* Pros:
  * Interactive, intuitive interface.
  * Explains what isn’t covered and gives detailed suggestions on how to increase code coverage. (Seems better than Codecov, but… it’s also a more refined product?).
  * Keeps a running history of your code coverage reporting, if desired.
* Cons / Callouts:
  * Appears to be "free to use," but Sonar wants you to store coverage data in their cloud and storage costs will apply unless you agree to host your own Sonar service -- in which case, we'll need to pay for a license. (Not ideal.)
  * Code coverage visuals, etc. are all viewed through the Sonar web front end, so the Sonar bot will make a comment with a direct link to the appropriate URL and users have to click that.
    * If Sonar is down, then we’re at their mercy unless we host Sonar ourselves – but then we have to support the hosted service and that’s not ideal.

## Code Coverage Badge: Options

### [shields.io](http://shields.io)

Compatibility:

* Codecov. View Codecov setup instructions [here](https://shields.io/badges/codecov-with-branch).
* SonarQube. View Sonar setup instructions [here](https://shields.io/badges/sonar-coverage).
* Coveralls. View Coveralls setup instructions [here](https://shields.io/badges/coveralls).

### Codecov

If we use Codecov for displaying coverage information, then we can utilize their built-in [status badge](https://docs.codecov.com/docs/status-badges) functionality that integrates with their coverage report tooling.

### Coveralls

Coveralls hosts status badges on their website if you prefer Coveralls' hosting over shields.io.

## Workflow Design

### Technical requirements

#### Bot Permissions

The bot should have the following accesses at a bare minimum, but may need more:

* GitHub comment
* S3 read permissions** (if the bot must be connected to S3 to read reports)

### Functional (Non-Technical) Requirements

#### Data Storage (for Storing Coverage reports)

If we’re not committing to hosting data through a 3rd party reporting service (like SonarQube), then code coverage reports should be uploaded to the cloud that supports one of pytest’s reporting formats (JSON, HTML, XML, lcov, or annotate).

* Users should not be able to view uploaded reports in the public cloud, as some services like AWS S3 can charge for data retrieval (depending on the tier you use).
* Sometimes coverage tools can also capture vulnerability/bug concerns, so if we change tooling in the future, we don’t want people seeing vulnerabilities/bugs.

We should generate a temporary, unique file path for each PR’s code coverage report so that PR reports don’t overwrite each other. (It’s okay to overwrite code coverage reports if someone creates multiple commits in the same PR, though.)

After PR is merged:

* Code coverage report for the PR along with its unique S3 path should be deleted.
  * Sometimes, contributor PRs have no merge conflicts but are still "X commits behind `main`" when merged, so we don’t want to use the PR’s code coverage report if it’s outdated.
* Regenerate code coverage report on `main` to update the code coverage badge.
* Upload the latest code coverage report in S3.

After PR is closed:

* Delete code coverage report for the PR along with its S3 path.

#### Bot Comment Contents

The bot should be able to make a comment that meets the high-level reporting requirements defined at the top of this doc (e.g., % coverage on new lines of code, etc.)

#### Coverage Badge

The coverage badge must:

* Report the latest code coverage % on main.
* Be placed at the top of the README and visible to all viewers.

### Additional Design Suggestions

#### Data Storage Cost Management Ideas

* The AWS S3 service scales storage automatically, so we can start with AWS’ [S3 intelligent tiering](https://aws.amazon.com/s3/storage-classes/intelligent-tiering/) to optimize for cost savings when uploading multiple PR coverage reports.
* We can use the [S3 Glacier storage class](https://aws.amazon.com/s3/storage-classes/glacier/) to save funds on data >1 week old (since we will likely only need it for historical reasons).
* **Side note**: Uploading to S3 will be helpful if we want to keep track of our code coverage progress, especially if we need to migrate to another tool for graphing.
