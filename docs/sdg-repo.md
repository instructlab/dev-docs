# New Repository Proposal: sdg

## Summary

This document proposes a new repository under the `instructlab` GitHub organization:

- `instructlab/sdg`

## Background

The `instructlab/instructlab` repository includes a basic implementation of
Synthetic Data Generation (SDG). This implementation does not implement the full
approach as described by the [LAB paper](https://arxiv.org/abs/2403.01081).

We desire to build out a more complete implementation of SDG that is more in
line with the LAB methodology. We propose a new repository to house this code
that publishes a new Python library called `instructlab-sdg`.  The reasoning for
a new repository and library includes:

- We expect multiple consumers of this code. The `ilab` CLI is one, but we also
  envision building a REST API around it to help support scaling out this
  functionality on a cluster.
- We expect there is broader community interest in an open-source library and
  service for synthetic data generation. We envision this library could support
  other data generation techniques over time.

## Maintainers

The initial team of maintainers for this repository will be a copy of the
`Backend Maintainers` GitHub team.

## Alternatives Considered

### Add to `instructlab/instructlab`

We could add this code to the existing `instructlab/instructlab` repository.

The primary argument against this approach is that we expect the scope of an
`instructlab-sdg` library to expand beyond the scope of what would be run by the
`ilab` CLI. We instead envision a different community of contributors organizing
around SDG specifically.
