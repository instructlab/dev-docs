# New repository for RAG

| Created  | Dec 5, 2024 |
| -------- | -------- |
| Authors | Bill Murdock |
| Replaces | N/A |
| Replaced by | N/A |

## What

We want a retrieval-augmented generation (RAG) capability that provides outstanding results with minimal effort, is seamlessly integrated with InstructLab, and is also general enough to be used in other applications as well.

## Why

Many InstructLab users want to train a model and then use it to RAG.  Often they build something simple themselves for this purpose.  Two problems with this approach:

- Building their own RAG is extra work.
- Users who are not experts on RAG might not build a RAG that provides outstanding results.

There is a very simple RAG capability at <https://github.com/instructlab/rag> .  It is not tightly integrated with InstructLab and it does not use any advanced RAG capabilities.  However, it might have some existing users so we can't just unilaterally delete it or replace it with something radically different.

## Goals

Provide a built-in alternative for users who do not want to build their own RAG.  Do not break the existing capability at <https://github.com/instructlab/rag> .

## Non-goals

Evaluation of RAG will be addressed in one or more other development documents.  That topic is out of scope for this document.

## Decision

- There will be a new repository for RAG.
- It will be located at <https://github.com/instructlab/retrieve>
- By mid-January, it will be available and working but not integrated with InstructLab.
- By mid-March, it will be integrated with InstructLab with the new repository being invoked by the core repository and maybe also by the SDG repository.
- Eventually, it will be integrated with InstructLab with the new repository being invoked only by the core repository.

## How

### By mid-January

- There will be a new repository for RAG.
- It will be located at <https://github.com/instructlab/retrieve>
- It will *not* be referenced by any other InstructLab repositories at that time.
- We will provide a sample Python notebook that *at least* shows how to use this repository to do both of the following:
  - After someone has run `ilab data generate`, the code in the notebook pulls extracted content from documents that were stored during the execution of that command and indexes that content in a vector database.  (Note that this will also require updates to the SDG code base to ensure that the extracted content is stored; currently only the legacy format Docling outputs are stored.)
  - The code in the notebook takes a session history plus a query and does both retrieval and response generation to produce answers (i.e., run-time RAG).  That code shows how to do this both with an unmodified open-source LLM for response generation *and* with an InstructLab fine-tuned LLM to compare before-and-after fine-tuning behavior.

### By mid-March

- The capabilities in `retrieve` will be improved with more advanced functionality.  (Details TBD)
- The InstructLab command-line interface will be updated as follows:
  - There will be one or more commands that can be configured to index content in a vector database.  Those commands will call out to `retrieve` to do this indexing.
  - The existing chat capability will be configurable to either use RAG (which would involve calling out to `retrieve`) or not use RAG (which would provide the current behavior).
- Both indexing and run-time RAG will be implemented by calls that go directly from the core repo (`instructlab/instructlab`) to  `retrieve` *unless* the work to migrate SDG preprocessing to the core repo is not complete by then (in which case there may need to be calls from SDG to `retrieve` until the migration completes).

### Beyond mid-March

- The capabilities in `retrieve` will continue to be improved with even more advanced functionality.  (Details TBD)
- Both indexing and run-time RAG will be implemented by calls that go directly from the core repo to `retrieve`.  The SDG repository will not directly invoke the retrieve repository since the document processing will all be done in the core repository and the outputs of that document processing will be consumed by both `retrieve` and SDG.

## Alternatives

- Put the indexing code in <https://github.com/instructlab/sdg> (SDG) and the run-time RAG code in <https://github.com/instructlab/instructlab> (core)
  - Pro: This has the advantage of not adding any new dependencies.
  - Pro: The document processing is already in SDG and chat functionality is already in core so this would require the fewest code changes.
  - Con: Splitting the RAG functionality across multiple repositories makes it more complicated to reuse in other applications outside of InstructLab.
  - Con: Many things we will want to do to add advanced functionality to make RAG more effective will require changes to both indexing and run-time RAG.  If those components are split across multiple repositories, that will make delivering such changes more complicated.
- Put the indexing and run-time RAG code in <https://github.com/instructlab/instructlab> (core)
  - Pro: This has the advantage of not adding any new dependencies.
  - Pro: However, since the existing document processing is in SDG, the flow for indexing for RAG would be a bit complicated (i.e., it starts with a CLI call handled by the core repo then goes to SDG for some of the document processing and then back to the core for vector database indexing).   That drawback will be eliminated if/when the document processing moves into the core repository.
  - Con: Putting the RAG functionality in the core repository requires any application that wants to use this functionality to bring in the entire core which then brings in all of the libraries it depend on, so this becomes an enormous dependency.  This would discourage reuse in other applications.
  - Con: Building a great RAG capability is a long-term grand challenge that will require a lot of dedicated investment.  That makes it a poor fit for a repository that also has a lot of existing responsibilities.
- Start by putting the code into existing InstructLab repositories (either of the above options) and then split if off into its own repository later.
  - Pro: Gets us integrated into InstructLab sooner.
  - Con: Adds extra work to the second phase where we have to split it off into its own repository.
  - Con: There is a risk that we never get around to splitting it off and we wind up stuck with the cons of being jammed in to other components indefinitely.
- Put the indexing and run-time RAG code in a new repo outside <https://github.com/instructlab/>.
  - Pro: This signals that this is not specific to InstructLab but is instead intended to be useful in a variety of applications.  That makes it more likely the work could have broader impact.
  - Con: If we put this out there as something that is intended to be useful in a variety of applications, the pressure is on us to make sure it is differentiated from other broadly applicable RAG capabilities.  Hopefully that will be true eventually, but it probably won't be true for a while.  It might make more sense to give this some time to mature as a local component of InstructLab before trying to spin it off as its own thing.
  - Con: If we put it out there as its own open source project, that project needs all of the infrastructure of a full open source activity (governing structures, communication tools and protocols, etc.).  That's a lot of work to set up.  Keeping it inside InstructLab for now lets us keep using the infrastructure that InstructLab has for this purpose).
  - Con: If we put it out there as its own open source project, it needs a name.  It is a lot of work to come up with a good name and there will be a lot of stakeholders with an interest in the name that comes up.
- Keep the indexing and run-time RAG code in <https://github.com/redhat-et/PaRAGon> which is an emerging technologies prototype for this work.
  - Mostly the same pros and cons as putting it in a new repo outside InstructLab plus the following:
  - Pro: A prototype for the code we want is already there.
  - Pro: It already has its own distinctive name (PaRAGon).
  - Con: The existing repository has its own simple command-line interface which is useful for the prototype but we don't want it in the capability we release because too many command-line interfaces will confuse users.
  - Con: The name PaRAGon seems fine to me, but probably more stakeholders need to weigh in on what a name would be.
  - Con: The `redhat-et` label suggests that this is something "owned" by Red Hat which makes sense for the prototype but not so much for something we want a community to own in the long run.
- Put the indexing and run-time RAG code in <https://github.com/instructlab/rag> AND keep the existing RAG functionality in that repository intact.
  - Pro: It already exists.
  - Pro: It avoids the confusion of having two different RAG repositories in <https://github.com/instructlab/>.
  - Con: It creates the confusion of having two different RAG solutions in the same repository.  I guess we could mitigate that with developer documentation and marking legacy stuff as "deprecated".
- Put the indexing and run-time RAG code in <https://github.com/instructlab/rag> AND eliminate the existing RAG functionality in that repository.
  - Pro: It already exists.
  - Pro: It avoids the confusion of having two different RAG repositories in <https://github.com/instructlab/>.
  - Pro: It avoids the confusion of having two different RAG solutions since we'd be eliminating the old one.
  - Con: That stuff must be there for a reason.  Maybe someone is using it.  It is not totally clear how we would even find out.  If someone can run this down maybe we can consider this option.

## Risks

- Adding a new repo and dependencies adds more continuous integration complexity and work.
- That extra work is why we're not planning calls from InstructLab core to the RAG capability until the mid-March time frame.  This risks missing some market opportunities.
- As noted earlier, putting the capability inside <https://github.com/instructlab/> signals that this is a component of InstructLab and not a generally useful feature.  That creates a risk that the work could miss out on additional opportunities for impact.  We hope to mitigate that risk by spinning it off to its own open source project when it is mature enough, but there is a risk that we will get distracted by other things and never get around to this.

## References

- <https://github.com/redhat-et/PaRAGon>
- <https://github.com/instructlab>
- <https://github.com/instructlab/rag>
