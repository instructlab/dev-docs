# Create A Repo for InstructLab Examples

## Idea Overview

Create a separate repository within the `instructlab` GitHub org called `examples`.
This repository would house [Jupyter notebooks](https://jupyter.org/) and other examples that illustrate parts of or the entire LAB methodology.
The `examples` repo would be a place where the maintainers of the InstructLab project can collect notebooks for the benefit of the community.
All notebooks submitted to this repository would be carefully documented and tested before being merged.

## Repository Structure

The repository will start by housing notebooks and have two categories of notebooks. Notebooks either live in the `combined-stages` or `use-cases` directories.

```bash
examples
|
|- notebooks
    |
    |- combined-stages
    |   |- training-with-eval
    |       |- requirements.txt
    |       |- training-with-eval.ipynb
    |- use-cases
    |   |- policy-documents
    |   |   |- requirements.txt
    |   |   |- legislative-act.ipynb
    |   |- instruction-manuals
    |   |   |- requirements.txt
    |   |   |- how-to-build-a-house.ipynb
```

### Notebooks for Combined InstructLab stages

Notebooks in the `combined-stages` directory go through parts of or the entire InstructLab workflow that users might want to reference or use.
Some examples of combined stages are a notebook that runs through training then evaluation or a notebook that goes from document pre-processing to SDG.

### Notebooks for End-to-End (e2e) use cases

Notebooks in the `use-cases` directory reflect real world use cases from start to finish.

## Additional Info

This repo would not be released as a package on PYPI but initially as just `.zip` and `.tar.gz` files on GitHub.
Releases would serve the purpose of giving users specific versions of notebooks they could run reliably.
