# GitHub Merge Strategy for InstructLab

This document describes the merge strategy used for Pull Requests within all repositories in the [InstructLab](https://github.com/instruct-lab) organization.

## Requirements for Merging a Pull Request

Every Pull Request that is made to an InstructLab repository should meet the below requirements - certain repositories such as [taxonomy](https://github.com/instruct-lab/taxonomy) may have additional requirements.

### Commit Messages and PR Description

PRs should be formatted as a series of logical commits that implement the
overall change proposed in the PR. Each commit should represent a logical step
in the progression of the change. The commit message for each should clearly
explain that step of the progression.

Here are a few examples of PRs that contained a series of commits:

- <https://github.com/instructlab/instructlab-bot/pull/125/commits>
- <https://github.com/instructlab/instructlab/pull/994/commits>
- <https://github.com/instructlab/instructlab/pull/951/commits>

Here are some external resources that provide guidance on writing good commit messages:

- <https://www.berrange.com/tags/commit-message/>
- <https://docs.kernel.org/process/submitting-patches.html#separate-your-changes>
- <https://github.com/kubernetes/community/blob/master/contributors/guide/pull-requests.md#smaller-is-better-small-commits-small-pull-requests>

While not a requirement right now, we may want to consider following a spec like
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) in the
future. It's worth a look for inspiration, at least.

When your PR contains multiple commits, it is helpful to reuse the work you put
into crafting high-quality commit messages. Here is a script you can use to generate
a summary of the commits in your PR. It starts with a list of the commits one
line at a time and then follows that with the full commit messages. The format works
for pasting directly into GitHub.

```sh
#!/bin/bash

git log --reverse --oneline origin/main..HEAD
echo
git log --reverse origin/main..HEAD
```

Here is an example PR that uses a PR description that was generated using this
script: <https://github.com/instructlab/sdg/pull/67>.

### CI checks

We should require that all CI checks pass on a Pull Request before it can be considered for merge. Every repository should have at mimimum the following checks:

- Linting
- Testing (Unit, Functional, etc)
- DCO Commit Signoff via a `Signed-off-by` header. There is a DCO check enabled for all repositories in this GitHub organization.

Additional checks might be required for repositories on a case-by-case basis.

### Approvals from Project Maintainers

At least one Project Maintainer should need to have an approving review on a Pull Request for it to be considered for merge. Requiring more reviews is left up to the discretion and consensus of the application maintainers team for a repository. The definition of a Project Maintainer can be found [here](https://github.com/instruct-lab/community/blob/main/governance.md#project-maintainers-overview).

Project Maintainers are given access permissions via [GitHub Teams](https://github.com/orgs/instruct-lab/teams) - you can see more details on the specific responsibilities of these teams [here](https://github.com/instruct-lab/community/blob/main/MAINTAINERS.md).

## Method for Merging a Pull Request

There are [three different merge methods offered by GitHub](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github) for merging Pull Requests.

We use the default merge method of creating merge commits for PRs. This is to ensure we retain the full commit history as intentionally structured by the PR author while also retaining metadata about the PR itself in the merge commit.

This requires project maintainers to include commit messages and the overall structure of the commit series as part of their review.

It is common that a PR author may need to do a final rebase to clean up their proposed commit series before a PR can be merged. It is also fine for a project maintainer to perform this step when the changes necessary are straight forward enough to do so.  This includes doing a final rebase on `main` if necessary. The PR itself should NOT include any merge commits of `main` back into the developer's branch. We expect the proposed commit series to be a clean set of commits against `main` without conflicts or merge commit history. We only use a merge commit to record the PR's inclusion into `main`.

## Merge Automation

Repositories may use [Mergify](https://mergify.io/) to automate the merge
process and enforcement of merge policies. Using this tool allows us to encode
the merge requirements in a file stored in the git repository itself. Once all
requirements are met, Mergify will automatically merge the PR.

An example configuration can be found in the [instructlab-bot
repo](https://github.com/instructlab/instructlab-bot/blob/main/.github/.mergify.yml).
