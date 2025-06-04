# Python Version Support

This document describes the project Python version support policy.

## Goals

1. Ensure that the project is compatible with latest Python releases.
2. Ensure that the project is compatible with the latest Python LTS releases.
3. Ensure reasonable choice of Python versions for users to choose from.
4. Ensure that CI and engineering resources are not wasted on supporting
   outdated Python versions.

## Approach

### Minimal Python version

The minimal Python version to support in all projects is determined based on
the availability of Python versions in popular operating systems (Fedora,
Debian, MacOS) and products built on top of the packages (RHEL AI).

At the moment of writing, the minimal Python version that all projects should
support is 3.11.

### Support multiple Python versions

To accommodate users that are not willing to upgrade to the latest Python
version, projects should strive to support several (upstream Python
versions)[https://devguide.python.org/versions/] in some capacity. At the very
least, all projects should run lightweight tests (unit, functional) against all
versions from the minimal supported version up to the latest version.

At the moment of writing, the versions to support are: 3.11, 3.12, 3.13.

### Dominant Python version

To concentrate resources on the most used Python version, the project should
pick one Python version that will be used to run all CI jobs. Other Python
versions may be supported in a more limited capacity (e.g. only running unit
tests).

In general, there's a single dominant Python version at any given time. During
adoption of a new dominant version, the project may have to temporarily support
two dominant versions at the same time. This is a temporary situation and plans
should be made to get back to the single dominant version as soon as possible,
to avoid wasting resources.

At the moment of writing, the dominant Python version is 3.11.

### Drop support for old Python versions

To avoid wasting resources on supporting outdated Python versions, projects
should proactively drop support for old versions below the minimal Python
version indicated above.

To drop support for a Python version, projects should remove all tests that are
specific to this version. They should also update `pyproject.yaml` metadata to
specify the minimal Python version equal to the one indicated above.
Documentation should be updated to reflect the new minimal version, if needed.

At the moment of writing, the versions to drop support for are: 3.10 or
earlier.

## Links

- [Upstream status of Python versions](https://devguide.python.org/versions/)
