# Modify Repository Proposal: Training

## Summary

This document proposes taking the existing [instructlab/training](https://github.com/instructlab/training) and transforming it into a library that will be consumed
by other clients such as the CLI and what is currently the `main_ds.py` file
that launches a full fine-tune.

## Background

Today, we have training implemented for different use-cases
existing in different repos. LoRA/QLoRA training currently lives
in the [`instructlab`](https://github.com/instructlab/instructlab)
CLI repo and contains logic for both Linux and MacOS training.
Each of these then brings additional logic for platform-specific accelerators.
For example NVIDIA CUDA and Intel Habana on Linux, and MLX and MPS on MacOS.

On the other hand, we have logic that exists today for full fine-tuning
in the [instructlab/training](https://github.com/instructlab/training) repo. Today, this implementation is CUDA-specific
as it relies on NCCL for handling communication across various
NVIDIA GPUs.
We want to bring support for alternative accelerators such as Intel Gaudi to this repo as well, without rewriting much of the logic.

We need to have a library which provides a common interface
for training, allowing all of the hard logic to be concentrated
in a single point.

The [instructlab/training](https://github.com/instructlab/training) repo should then
become the home for the overall training library.

This library will then contain a "simple" interface for all
other clients to pull from and use without much needing to change
on their side other than how they intend to use it.

## Maintainers

The maintainers should be the folks who currently work on training.
Namely:

- [Aldo Pareja](https://github.com/aldopareja)
- [James Kunstle](https://github.com/orgs/instructlab/people/JamesKunstle)
- [Oleg Silkin](https://github.com/orgs/instructlab/people/RobotSail)
- [Mustafa Eyceoz](https://github.com/orgs/instructlab/people/Maxusmusti)

We can also blanket access by setting it to the `Backend Maintainers` GitHub team.

## Alternatives Considered

### Keep everything separate

Rather than consolidating the logic, we can keep the existing
logic as-is.

This would mean that the [CLI repo](https://github.com/instructlab/instructlab) and the [training repo](https://github.com/instructlab/training) would both maintain their own implementations of training.

This means that if extra logic must be added for Intel Gaudi or NVIDIA, it would need to be added and tested in two different places.

Since we need to move the full training to live inside of the the
[CLI repo](https://githhub.com/instructlab/instructlab),
this would now have two duplicate implementations of the
full fine-tune training and add additional points of complications.

### Move everything into one repo but don't design an interface

We can move the existing training logic into the training repo
and simply have the existing clients consume it this way.

The challenge here is that a lot of the logic is very specific
to the client application, so there would be cross-development.

If someone wants to create a new PR to the CLI repo for a change
they're making to the LoRA training, they'd need to create
another PR into the training repo and maintain two PRs at once.
When a maintainer requests a change in one PR, both would need to be updated to accommodate each other.

The challenges this presents to both the developers and maintainers
is clear. Therefore a natural conclusion is that we need a separate library that provides an extensible interface for other clients to consume.