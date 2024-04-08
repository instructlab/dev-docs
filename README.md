# InstructLab Architecture

This repository contains developer docs and design artifacts that define the
architecture and design of cross-component interactions in the Instruct Lab
project. The repo also serves as a home for technical policies that apply
across all components.

## Governance

Governance for InstructLab overall is documented in the [community
repository](https://github.com/instruct-lab/community/blob/main/governance.md).

There is not (yet) a group that provides technical oversight across all of
InstructLab. If such a group exists in the future, that group would assume
ownership over the contents of this repository, with input from maintainers of
all components within InstructLab.

In the meantime, ownership of this repository is shared collectively by the
maintainers of all components within InstructLab. Required approval is loosely
defined and depends on the scope of each proposal. In general, maintainers from
all affected components should be sought for approval. The broader the scope or
more controversial the topic, the more broad consensus is required to proceed.
We expect the collective group of maintainers to use their best judgment to
decide what constitutes appropriate approval on a case-by-case basis.

## Formatting Guidelines

Design documents should be placed in `docs/`.

Developer docs should be placed in `dev-docs/`.

### Text

Files should be in [Markdown](https://github.github.com/gfm/) format.

### Images

Diagrams are encouraged, but must be submitted in a format where they can be
easily updated in the future as needed. Some options include:

* [Mermaid](https://github.com/mermaid-js/mermaid#readme)
* [Excalidraw](https://excalidraw.com/)
** Be sure to leave "Embed Scene" turned on when exporting the PNG.
