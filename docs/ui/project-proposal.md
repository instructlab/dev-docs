# New Repository Proposal: ui

## Summary

This document proposes a new repository under the `instructlab` GitHub organization:

- `instructlab/ui`

## Background

Currently we don’t have public repository under InstructLab that hosts UI-related work. The goal of this project is to provide a space for hacking a UI for the InstructLab. The scope of this project is to support the following personas through the UI:

- External contributors who want to contribute skill and knowledge to the taxonomy repo
- Triagers who will be evaluating the taxonomy contributions.

These personas are not limited to the above list, but the initial focus will be on these two.

Intent is to build this UI as a SaaS service for the upstream project and its taxonomy repo, but build in a way that someone could deploy the same service on their own infrastructure for managing their own taxonomy repo.

## Maintainers

The initial team of maintainers (GitHub Team - `UI Maintainers`) for this repository will be:

- Anil Kumar Vishnoi <avishnoi@redhat.com>
- Brent Salisbury <bsalisbu@redhat.com>
- Taiga Nakamura <taiga@us.ibm.com>
- Guang-Jie Ren <gren@us.ibm.com>
- Juan Cappi <jmcappi@us.ibm.com>
- Daniel Tan <chungtan@us.ibm.com>
- Gregory Pereira <grpereir@redhat.com>

## Seed Code Contribution

We are planning to seed this repository with the code from the `instructlab/instructlab-bot` repository [here](https://github.com/instructlab/instructlab-bot/tree/main/ui). This code will be used as a starting point for the UI work. This code is already open source and licensed under the Apache 2.0 license.

## Alternatives Considered

### Use `instructlab/instructlab-bot`

We currently have a very initial version of InstructLab UI related code present in `instructlab/instructlab-bot` repo (in "ui" directory). We can continue hacking the code in that repo, but I believe that is not the right place for the UI work, as instructlab-bot is supposed to be a backend component that is scoped to work as a triagers assistant tool with no direct interfacing to contributors or traigers. To work with a cleaner scope of these separate work streams, it makes sense to separate the UI work in its own repository.
