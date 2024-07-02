# New Repository Proposal: eval

## Summary

This document proposes a new repository under the `instructlab` GitHub organization:

- `instructlab/eval`

## Background

The `instructlab/instructlab` repository currently includes no real implementation
of Evaluation as described by the [LAB paper](https://arxiv.org/abs/2403.01081). The
closest implementation currently in `instructlab/instructlab` via the `ilab test` command.

`ilab test` as of this writing is only implemented for macOS with M-series chips. It uses
a JSON Lines file and a LoRA adapter to compare output of a given model before and after
LoRA training with MLX, thus the macOS M-series dependency.

We desire to build out a library for methods that satisfy the evaluation described in the
paper, using more high-level evaluation schemes such as
[Multi-turn Benchmark](https://arxiv.org/abs/2306.05685) for skills and
[Massive Multitask Language Understanding](https://arxiv.org/abs/2009.03300) (MMLU) for
knowledge. We propose a new repository to house this code that publishes a new Python
library called `instructlab-eval`. The reasoning for a new repository and library includes:

- We expect multiple consumers of this code. The `ilab` CLI is one, but we also envision
building a REST API around it to help support scaling out this functionality on a cluster.
- We expect there is broader community interest in an open-source library and service for
evaluation. We envision this library could support other evaluation techniques over time.
- We also realize that much of model evaluation is generally useful outside the context of
InstructLab. Other libraries may emerge in the broader ecosystem that handle parts of what
we need, while this library will always remain to handle the InstructLab-specific details
of how evaluation works in our workflow.

## Maintainers

The initial team of maintainers for this repository will be a copy of the
`Backend Maintainers` GitHub team.

## Alternatives Considered

### Add to `instructlab/instructlab`

We could add this code to the existing `instructlab/instructlab` repository.

The primary argument against this approach is that we expect the scope of an
`instructlab-eval` library to expand beyond the scope of what would be run by the
`ilab` CLI. We instead envision a different community of contributors organizing
around Evaluation specifically.
