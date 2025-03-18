# Create Separate Repo for User Utilities

## Idea Overview

Create a separate repository within the `instructlab` GitHub org called `support-utils`.
This repository would house scripts and notebooks outside of the scope of the LAB Methodology implemented in the [Instructlab Core](https://github.com/instructlab/instructlab) repository that enhance the InstructLab experience.
Many users and community members already have such scripts they use day to day.
The `support-utils` repo would be a place where the maintainers of the InstructLab project can collect and curate them for the benefit of the community.
Scripts in this repository may become features or incorporated in the InstructLab Core repository after use and review by users and developers.

## Repository Structure

The repository will have two categories of scripts. Scripts either live in the `hack` and `beta` directories.

```bash
support-utils
|
|
|- beta
|
|- hack
```

The `hack` directory is open for the contribution of scripts of any quality.

Scripts in the `beta` directory will be required to have documentation, and automated functional testing.
These scripts are meant to be run by users for feedback and may graduate into full blown features in other InstructLab repos.

Beyond this initial structure, the structure within those two directories will evolve as scripts are contributed to each.

## Additional Info

A few areas of focus for the first scripts that will be added to the repository are:

- Automating qna.yaml creation
- Assessing document readiness knowing the limitations of Docling
- Visualizing synthetically generated data for inspection

This repo would not be released as a package on PYPI but initially as just `.zip` and `.tar.gz` files on GitHub.
Releases would serve the purpose of giving users having specific versions of scripts in `beta` and for development project management purposes.
