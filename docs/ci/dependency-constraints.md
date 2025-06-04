# Dependency Constraints in InstructLab CI

This document describes how Python dependencies should be managed in
InstructLab CI.

## Goals

1. Ensure that the CI environment is consistent and reproducible.
2. Ensure that new dependency releases do not break the CI environment.
3. Ensure that we adopt new dependency releases in the CI environment in a
   timely manner.

## Approach

### Pin all versions with constraints files

Each repository should contain a [pip
constraints](https://pip.pypa.io/en/stable/user_guide/#constraints-files) file
that lists the pinned versions of all dependencies used in the CI environment.

In case a repository supports multiple platforms (`linux` vs `darwin`) or
accelerators (`cuda`, `cpu`, `hpu`), multiple constraints files may be
generated.

The constraints files should be used in all CI jobs that install Python
dependencies (using `pip` or otherwise), including linters, unit tests,
integration tests, and functional tests.

### Update constraints files regularly

Each repository should define a new `tox` target called `constraints` that will
be used to re-generate all the constraints files in the project.

Updates to these files should be generated automatically by the CI system using
the [update-constraints](https://github.com/instructlab/ci-actions/tree/main/actions/update-constraints)
action from `ci-actions` repository and should not be modified manually
(subject to rare exceptions). A periodic CI job should be added to ensure this
happens on a regular basis (at least once a week).

The job will update constraints file and post the result as a PR for review.
The PR will be validated by all the relevant CI jobs. Project core team is
expected to review these PRs in a timely manner (within 3 business days). The
team should make sure that all the relevant CI jobs are passing before merging
the PR.

### Uncap all dependencies

At this point, no new dependency releases should affect the CI environment.
Projects should then uncap all their dependencies in requirements files, as per
[dependency management policy](../dependency-management.md).

Note: In rare situations, a cap may be justified. For example, when we know for
sure that a new release of the upstream dependency will break the project, and
when we don't have capacity to deliver a fix in a timely manner. These
situations should be rare and a mitigation plan should be in place to uncap the
dependency.

Specifically,

- A tracking issue should be reported in the issue tracking system capturing
  any known details about the issue.
- A new **temporary** constraint should be added to `constraints-dev.txt.in`
  file. The temporary constraint should refer to the tracking issue in a
  comment above it.
- The tracking issue should be assigned a high priority and considered a
  blocker for an upcoming release.
