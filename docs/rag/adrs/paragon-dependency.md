# InstructLab RAG will depend on the open-source project currently known as PaRAGon

## Context

[PaRAGon](https://github.com/redhat-et/PaRAGon) is a software asset from the Red Hat Emerging Tech team.  That team intends to make it a full open source project with its own governance, community engagement, contribution models, etc.  They are open to partnering with other teams that have more experience with open source as well.  There is a [draft pull request](https://github.com/instructlab/instructlab/pull/2736) for adding functionality to InstructLab.  That pull request is largely inspired by PaRAGon, but is not actually invoking PaRAGon code.   We intend to merge some version of that pull request or something similar so we can get an initial implementation available to users right away.  However, it is not really a sustainable long-term development model to maintain our own code base that is largely inspired by PaRAGon since it would require us to constantly be reading all of the new code that goes into PaRAGon and figuring out how to do something equivalent in our own code base to keep up with the new developments there.  So we'd like to move as quickly as possible to a model where the essential RAG functionality exists only in PaRAGon and InstructLab links to it as a dependency.  This is how InstructLab interoperates with Docling and countless other dependencies.

Note that the name PaRAGon is preliminary and may change to avoid conflicts with third-party trademarks.  This document will continue to refer to the asset as PaRAGon below, but when it does so it really means "the asset that is currently called PaRAGon but might be called something different in the future".

### Alternatives

- InstructLab could follow the model in the draft pull request indefinitely.  That would give InstructLab more flexibility to focus on InstructLab-specific needs.  However, it would also lead to a lot of duplicated effort with the team working on PaRAGon, which already pays a lot of attention to InstructLab-specific needs.
- InstructLab could adopt an off-the-shelf RAG platform instead of PaRAGon.  For example, PaRAGon itself is built on Haystack; we could just depend on Haystack directly.  However, Haystack is an very flexible platform for developers, so adopting Haystack directly in InstructLab would entail making a lot of technical decisions that have considerable impact on the effectiveness of the solution.  So either we would make the same decisions that PaRAGon makes (which brings us back to the option above) or we would make different decisions (which would mean even more duplicated effort plus substantial inconsistency).
- InstructLab could fork its own copy of PaRAGon and put that in the InstructLab organization in github (either in an existing repository there or a new one).  That would make it a lot easier to incorporate changes from PaRAGon than just having code inspired by PaRAGon, because we could just copy the entire code-base over when we want to update the fork.  However, it still seems a lot clunkier than just having a dependency on PaRAGon.

## Decision

- InstructLab will initially have its own RAG implementation that is loosely inspired by PaRAGon.
- However, once PaRAGon is fully open source and with proper governance, community engagement, etc., InstructLab will remove its own implementation and instead depend on PaRAGon.
- Once this is complete, the RAG-specific code in InstructLab will be limited to direct call-outs to PaRAGon and/or manipulation of InstructLab-specific data structures (e.g., InstructLab taxonomies) to convert data into a form that PaRAGon can process.

## Consequences

- PaRAGon is intended to be a common RAG solution for a variety of capabilities: InstructLab, [RamaLama](https://github.com/containers/ramalama), etc.  Since InstructLab will depend on it, the RAG behavior for InstructLab will be consistent with the RAG behavior for these other capabilities.  This will improve the experience for end-users who want to integrate and/or move between these capabilities.
- If and when InstructLab developers want to contribute functionality that impacts the behavior of the RAG in InstructLab, they will need to contribute these changes to PaRAGon.  The PaRAGon maintainers will have ultimate authority over these changes.
