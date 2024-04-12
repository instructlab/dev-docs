# GitHub Merge Strategy for InstructLab

This document describes the merge strategy used for Pull Requests within all repositories in the [InstructLab](https://github.com/instruct-lab) organization.

## Requirements for Merging a Pull Request

Every Pull Request that is made to an InstructLab repository should meet the below requirements - certain repositories such as [taxonomy](https://github.com/instruct-lab/taxonomy) may have additional requirements.

### CI checks

We should require that all CI checks pass on a Pull Request before it can be considered for merge. Every repository should have at mimimum the following checks:
- Linting
- Testing (Unit, Functional, etc)
- DCO Commit Signoff via a `Signed-off-by` header. There is a DCO check enabled for all repositories in this GitHub organization.

Additional checks might be required for repositories on a case-by-case basis.

### Approvals from Project Maintainers

At least one Project Maintainer should need to have an approving review on a Pull Request for it to be considered for merge. The definition of a Project Maintainer can be found [here](https://github.com/instruct-lab/community/blob/main/governance.md#project-maintainers-overview).

Project Maintainers are given access permissions via [GitHub Teams](https://github.com/orgs/instruct-lab/teams) - you can see more details on the specific responsibilities of these teams [here](https://github.com/instruct-lab/community/blob/main/MAINTAINERS.md).

## Method for Merging a Pull Request

There are [three different merge methods offered by GitHub](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github) for merging Pull Requests.

We use the default merge method of creating merge commits for PRs. This is to ensure we retain the full commit history as intentionally structured by the PR author while also retaining metadata about the PR itself in the merge commit.

This requires project maintainers to include commit messages and the overall structure of the commit series as part of their review. When multiple commits are present, they should represent a logical series of changes that implement the overall change proposed in the PR. The commit message for each should clearly explain that step of the progression.

It is common that a PR author may need to do a final rebase to clean up their proposed commit series before a PR can be merged. It is also fine for a project maintainer to perform this step when the changes necessary are straight forward enough to do so.  This includes doing a final rebase on `main` if necessary. The PR itself should NOT include any merge commits of `main` back into the developer's branch. We expect the proposed commit series to be a clean set of commits against `main` without conflicts or merge commit history. We only use a merge commit to record the PR's inclusion into `main`.

