# InstructLab Developer Documentation

This repository contains design artifacts that define the architecture and
design of cross-component interactions in the InstructLab project. The repo
also serves as a home for technical policies that apply across all components.

## Governance

Governance for InstructLab overall is documented in the [community
repository](https://github.com/instructlab/community/blob/main/governance.md).

The [InstructLab Oversight
Committee](https://github.com/instructlab/community/blob/main/MAINTAINERS.md) is
responsible for the contents of this repository.

The rules for merging depend on the type of change in question and its scope of impact.

* Trivial changes may be merged with 1 review from any InstructLab maintainer.
* Non-trivial changes have more loosely defined requirements. Input should be sought
  out from maintainers of relevant components. The broader the scope or more
  controversial the change, the more broad the consensus should be required for
  merging. The final approval and merge falls to a member of the Oversight Committee.
  This final review is to ensure that adequate opportunity and attention has
  been given by the affected parties.
* Any maintainer or oversight committee member may request that a change receive
  a full vote from the Oversight Committee. More substantial policy changes or a
  proposed new project under InstructLab are examples of when this may be
  appropriate.

## Formatting Guidelines

Design documents should be placed in [docs/](./docs). API Specifications should be placed in [api-definitions/](./api-definitions).

### Text

Files should be in [Markdown](https://github.github.com/gfm/) format.

### Images

Diagrams are encouraged, but must be submitted in a format where they can be
easily updated in the future as needed. Some options include:

* [Mermaid](https://github.com/mermaid-js/mermaid#readme)
* [Excalidraw](https://excalidraw.com/)
** Be sure to leave "Embed Scene" turned on when exporting the PNG.

### API Specifications

API definitions use [OpenAPI](https://www.openapis.org/) format in [YAML](https://yaml.org/).
