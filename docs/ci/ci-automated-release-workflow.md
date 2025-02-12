# Creating Automated Releases: Design Document

## Motivation & Overview

Presently, the release processes for every library within this `instructlab` GitHub organization is entirely manual:

For example, for a typical y-stream release, a maintainer has to:

1. Manually create a new release branch -- e.g., `release-0.y.0`,
2. Manually create a pull request against `release-0.y.0` to cap the versions on some of the dependencies defined in their library's `requirements.txt` file,
3. Optionally trigger an E2E test against that pull request,
4. Wait for all pull request CI checks to complete,
5. Manually request two maintainers to approve the pull request, and
6. Manually create a release from the GitHub UI using that new branch

This entire process takes at least 10 minutes of manual work, plus however long it takes for the pull request's checks to complete. (In some repositories, like the core repo, this can take 2+ hours.)

Going forward, we should automate these release processes so that contributors and maintainers can focus more on development work and less on creating actual releases.

## Generic Automation Workflow: Major Releases, Minor Releases, and Z-Stream Releases

### Brief Overview of the Automation

The automation logic described in this dev-doc will be published in the form of an in-house GitHub action called `create-automated-release` and it will therefore be callable from any workflow file. Thus, it is strongly recommended that each repository maintainer creates a `.github/workflows/automated-release.yml` workflow file to call this `create-automated-release` GitHub action from.

### Goals of the Automation

The automated process should be:

1. Configurable so that any library maintainer can configure the automation to meet their specific repository's needs
2. Schedulable so that releases are generated according to a specific cadence

Scheduled releases are generally important for setting expectations around release cadences, but they are by no means required for every library and may not even be applicable to some in this GitHub organization.

## Y-Stream Automation Workflow

### Overview of Y-Stream (Minor) Release Automation

Y-stream (minor) releases have historically been handled differently from Z-stream releases. Z-stream releases oftentimes involve backports for bugfixes and may require manual code rebasing to get those backports merged into the appropriate existing release branch. Therefore, we can think of Y-stream release logic as the "basis" for Z-stream release logic, which takes the Y-stream logic and builds upon it to account for backporting and other desirable actions.

### Configurable Components

As mentioned above, there are configurable components within this release process automation. The diagram in the next section references two configurable components:

#### Trigger Schedule

The trigger schedule defines the day and time (in UTC) when the release process will kick off.  This schedule can be disabled if desired, and maintainers can trigger the release process manually when needed instead.

As mentioned in the brief overview of the automation, this in-house `create-automated-release` action will be callable from any workflow file. Thus, each library maintainer who wants to create a scheduled release should first create a  `.github/workflows/automated-release.yml` workflow file in their repository and define a schedule for it. For example:

```yaml
name: Create Y-Stream Release

on:
  schedule:
    - cron: '30 1 1,15 * *' # Triggers at 1:30am UTC every 2 weeks on the 1st and the 15th day of each month
```

#### Custom List of Dependencies to Cap

With each release, some library maintainers may want to cap the version of certain dependencies within their `requirements.txt` file as well as specify the desired upper cap for each one.

The list of dependencies to cap should be provided in a file called `automated-release-config.yml`. This configuration file may be expanded in the future to accommodate more configurations as needed. Example:

```yaml
y_stream_release:

  dependency_caps:
    enable: true # optional parameter. If set to false, then none of the dependencies in `requirements.txt` will be capped.
    packages:
      instructlab-sdg: "+0.1.0"
      instructlab-eval: "+0.2.0"
      instructlab-training: "+1.0.0"
```

The keys specify the dependencies to cap. (In this case, we only have three: `instructlab-sdg`, `instructlab-eval`, and `instructlab-training`. The other dependencies in the `requirements.txt` file will be ignored and left untouched.)

The value for each key specifies the cap relative to the current lower bound. For example, let's say the current `requirements.txt` file looks like this:

```bash
instructlab-sdg>0.20.0
instructlab-eval>0.1.0
instructlab-training
```

In this case, the automated logic will create a pull request that updates the `requirements.txt` file like so (ignoring the inline explanations I provided as comments):

```bash
instructlab-sdg>0.20.0,<=0.21.0 # increment by 0.1.0 because of the '+0.1.0' in the `cap-deps.cfg` file
instructlab-eval>0.1.0,<=0.3.0  # increment by 0.2.0 because of the '+0.2.0' in the `cap-deps.cfg` file
instructlab-training            # do nothing because there was no lower bound set.
```

If `dependency_caps` is not defined or `enable` is set to `False` under the `dependency_caps` key, then none of the dependencies defined in `requirements.txt` will be capped. If a dependency to cap is defined in `dependency_caps` and that dependency doesn't exist, then it will be ignored.

### Example Workflow File

Below is an example workflow file used to call this in-house GitHub action:

```yaml
name: Create Y-Stream Release

on:
  # Run every Monday at 1AM UTC
  schedule:
    - cron: '0 7 * * 1 ' 

  # Allow manual dispatch, too
  workflow_dispatch:
    inputs:
      pr_or_branch:
        description: 'pull request number or branch name'
        required: true
        default: 'main'

jobs:
  create-release:
    runs-on: ubuntu-latest
    steps:
      - name: "create-automated-release"
        uses: instructlab/ci-actions/create-automated-release@v1
        with:
          release_config: ".github/automated-release-config.yml" # points to where the library's release config is located in its repository
```

### Y-Stream Release Flow Diagram

![Automated workflow for creating new GitHub releases](../images/design-diagram-for-automated-releases.png)

## Z-Stream Release Automation Workflow

### Overview of Z-Stream Release Automation

To be added at a later date.