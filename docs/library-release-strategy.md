# Library Release Strategy for InstructLab

This document describes the overarching release strategy and policies for Python libraries in the InstructLab organization.

## Background and Problem Statement

The InstructLab organization features multiple code repositories that are tagged and released as Python libraries.
Primarily this has been the [CLI repository](https://github.com/instructlab/instructlab) but has grown over time,
including but not limited to:

- [sdg](https://github.com/instructlab/sdg)
- [training](https://github.com/instructlab/training)
- [eval](https://github.com/instructlab/eval)
- [schema](https://github.com/instructlab/schema)
- [GPTDolomite](https://github.com/instructlab/GPTDolomite)
- [instructlab-quantize](https://github.com/instructlab/instructlab-quantize)

You can see the published versions of these libraries on PyPI [here](https://pypi.org/search/?q=instructlab).

While these libraries are all owned and maintained by the InstructLab organization, only the CLI has an official
[release strategy](https://github.com/instructlab/instructlab/blob/main/docs/release-strategy.md) documented. Other library
releases have been handled directly by the Maintainer teams at their own discretion. The organization needs to have certain
overarching principals around this topic, while still allowing for flexibility for each library on case-by-case basis.

## Proposal

By default, each existing and new library should have the following a `release-strategy.md` aligned with the CLI doc. This proposal recognizes that certain libraries may need flexibility
on case-by-case basis - therefore, Maintainer teams are empowered to modify these documents as they see fit, so long as the
following tenants remain consistent:

1. Packages **must** be named `instructlab-<package-name>`
1. Packages **must** follow the `X.Y.Z` numbering scheme (i.e. [semvar](https://semver.org/))
1. Packages **must** have GitHub tagged releases named `vX.Y.Z`
1. Packages **must** use release branches for Y-Streams of the form `release-vX.Y`
1. Packages **must** maintain a `CHANGELOG.md`
1. Maintainer teams **must** publicly communicate Y-Stream releases through official InstructLab channels such as Slack or Mailing Lists. Z-Stream release communication is up to Maintainer discretion.
