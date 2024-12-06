# The Road to 1.0.0

_Or: How I Learned to Stop Worrying and Love to GA_

## Context and Goals

The `instructlab/instructlab` repo started off as `instructlab/cli` - a basic Python Click-based command-line interface designed to prototype an application capable of
running the LAB methodology created by IBM Research. As the project evolved and the organization looked into creating a proper PyPI package for it, the decision was made
to rename the repo to `instructlab/instructlab` to keep the repo name consistent with the PyPI package name. The rest of this document will being using "InstructLab" to
refer to this repo and Python package.

Today, InstructLab has gone from a scrappy research project to an upstream community serving as the basis for multiple downstreams, with the goal
to continuing to evolve the community to encourage more participation from additional stakeholders. To wit, it would behoove us to determine what exactly we should be
roadmapping between now and a proper 1.0.0 release, which demonstrates the following to existing and potential community members:

1. An official goalpost for the community denoting the evolution of InstructLab from a pre-1.0 project lacking the stability and supportability typically seen from 1.0-and-beyond projects.
1. A dedicated set of V1 interfaces, both for internal configs and an API, that can be counted on for continuous usage of InstructLab 1.0 with future provisions made for backwards compatibility for subsequent Y-Streams and Z-Streams.
1. A commitment from the Oversight Committee and Maintainer teams to continue to maintain InstructLab throughout a 1.y cycle and work towards an eventual 2.0.

## MVP for an InstructLab 1.0.0

At a high-level, these are the items the Maintainer teams believe should serve as prereqs for releasing an InstructLab 1.0.0:

### Updating relevant references of "CLI" to "Core"

As noted in the `Context and Goals` section, InstructLab started off as just as a CLI - however, we are planning for this package to serve as a more general "Engine" -
being a place where a future REST API can be defined that is used by both the CLI aspect as well as an official GUI for orchestrating the entire LAB workflow. Despite
this, the repo is often still referred to as "the CLI". We as an organization need a better term to refer to this repo as, and should adopt the relevant documentation
and meetings accordingly.

An open community vote made as part of the drafting of this document decided that "Core" would be the new term used. You can see a record of the vote
[here](https://github.com/instructlab/dev-docs/pull/159#issuecomment-2514885516). This name change will begin to go into effect after the merging of this document
and should be completed by the time of a 1.0.0.

### A fully-realized configuration scheme, centered around the usage of system profiles

The InstructLab configuration scheme has transformed in many ways since the project's inception, from the `config.yaml` file that initially served as the user's config,
to the addition of code-based Pydantic defaults, to train profiles, to system profiles. We need to fully-decouple this config from the Click library, remove the need for
a `config.yaml` file, and have a consistent config scheme that can be easily extended.

### An official v1 REST API schema

We need to have a defined v1 REST API schema - while this does not preclude future updates, something mature enough to serve as a v1 API throughout subsequent Y-Streams
for an InstructLab 1.0 is a must for such a milestone.

### Integration of InstructLab with RAG

RAG is currently being planned on being integrated into InstructLab - that work should be in a stable state adhering to our v1 API standard.

### An upgrade path to subsequent Y-Streams and an eventual 2.0

Any user wishing to install an InstructLab 1.0 must have an upgrade path to 1.1, 1.2, ..., 1.n. Upon being ready for an InstructLab 2.0, we should also be expecting to
provide a path for users wishing to upgrade from our final 1.y stream to 2.0.

### Backwards compatibility across the 1.y stream

Any user going down our upgrade path described above should expect that the release they upgrade to is backwards-compatible with the release they upgrade from.

### An official hardware support matrix

We need to have a documented matrix of what hardware footprints we support and to what extent - this includes hardware we know will not work, hardware that we know might
work, and hardware we have confirmed will work with regular CI testing.

### A robust CI ecosystem

We should have a CI ecosystem that includes linting as well as unit, functional, and integration/end-to-end (E2E) tests in the InstructLab repo, along with proper documentation and Makefiles that allow developers to easily run subsets of them locally on their machines.

## Q&A

**Q. What about the libraries? Will they 1.0.0 as well?**

A. It depends - we historically have not aligned the InstructLab and Library releases on a particular version numbering scheme, apart from matching Y-Streams to Y-Streams (e.g., InstructLab 0.20 used SDG 0.4, Training 0.5, and Eval 0.3). At this stage, this document scopes only the prereqs we want for the InstructLab package.

## Conclusions and Decision Outcome

This document will be debated and updated as part of the Pull Request review process. Upon reaching a lazy consensus by the Oversight Committee and Maintainer teams, the author of this document (Nathan Weinberg) will merge the document, denoting the following:

1. The items in the above section `MVP for an InstructLab 1.0.0` will become official prerequisites for the InstructLab CLI Maintainer team to releasing a `1.0.0` of the InstructLab.
2. Any amendments to this list can only be made with a subsequent PR editing this document, subject to the same review process.
