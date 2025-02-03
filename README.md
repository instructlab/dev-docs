# InstructLab Developer Documentation

This repository contains design artifacts that define the architecture and
design of components in the InstructLab project. The repo also serves as a home
for technical policies that apply across all components.

## Governance

Governance for InstructLab overall is documented in the [community
repository](https://github.com/instructlab/community/blob/main/GOVERNANCE.md).

The [InstructLab Oversight
Committee](https://github.com/instructlab/community/blob/main/MAINTAINERS.md) is
responsible for the contents of this repository.

The rules for merging depend on the type of change in question and its scope of impact. If you
are unsure about the scope of impact for a change, i.e. if a change is trivial or non-trivial,
please ping the Oversight Committee for help.

* Trivial changes may be merged with 1 review from any InstructLab maintainer.
  * Examples of trivial changes include minor wording adjustments or typo fixes in
    documentation, changes to CI fixes, CI dependency updates, etc.
* Non-trivial changes have more loosely defined requirements. Input should be sought
  out from maintainers of relevant components. The broader the scope or more
  controversial the change, the more broad the consensus should be required for
  merging. The final approval and merge (or action, e.g. deleting a repo)
  falls to two maintainers of any InstructLab Organization repository as well as
  an additional third maintainer of any InstructLab Organization repository to
  merge the PR after verifying that sufficient reviews have been given. If there are
  disputes on the design document that cannot be resolved, an Oversight Committee
  member can be consulted as an arbitrator. These approvals ensure that
  adequate opportunity and attention has been given by the affected parties.
  * Examples of non-trivial changes include approving proposal for new repositories,
    creation of new repositories, changes to organization level GitHub settings, archiving
    or deleting repositories, design proposals, etc.
* Any maintainer or oversight committee member may request that a change receive
  a full vote from the Oversight Committee. More substantial policy changes or a
  proposed new project under InstructLab are examples of when this may be
  appropriate.

## Formatting Guidelines

Design documents should be placed in `docs/`.

### Text

Files should be in [Markdown](https://github.github.com/gfm/) format.

### Images

Diagrams are encouraged, but must be submitted in a format where they can be
easily updated in the future as needed. Some options include:

* [Mermaid](https://github.com/mermaid-js/mermaid#readme)
* [Excalidraw](https://excalidraw.com/)
** Be sure to leave "Embed Scene" turned on when exporting the PNG.
