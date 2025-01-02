# Code location for RAG

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

There is a very simple RAG capability at <https://github.com/instructlab/rag> .  It is not tightly integrated with InstructLab and it does not use any advanced RAG capabilities.  However, we have a request from a stakeholder to not just unilaterally delete it or replace it with something radically different.

## Goals

Provide a built-in alternative for users who do not want to build their own RAG.  Keep the existing capability at <https://github.com/instructlab/rag> somewhere, but potentially somewhere other than it is now (e.g., in a new branch of the existing repository).

## Non-goals

Evaluation of RAG will be addressed in one or more other development documents.  That topic is out of scope for this document.

## Decision

- For now, RAG will be located in the core repository in its own directory: `src/instructlab/rag` in the core InstructLab repository (<https://github.com/instructlab/instructlab>).

## How

### Phase 1

- RAG will be located in the core repository in its own directory: `src/instructlab/rag` in the core InstructLab repository (<https://github.com/instructlab/instructlab>).
- This directory will include all of the following:
  - Loading the content from Docling-format JSON files (that are produced by SDG preprocessing).
  - Chunking that content to sizes that fit the requirements of the selected embedding model for vector database storage and retrieval.
  - Storing those chunks with their vector representations in a vector database.
  - End-to-end runtime RAG.  The initial version of this includes the following:
    - Taking as input a session history (including a current user query) and providing a response (e.g., something along the lines of the [OpenAI chat completion API](https://platform.openai.com/docs/api-reference/chat/create)).
    - During that processing, it retrieves relevant search results from the vector database, it converts those into a prompt to send to the response generation model, it prompts that model, and it returns the response from that model.
- This will be invoked from the existing `ilab` CLI, as described in the [RAG ingestion and chat pipelines](https://github.com/instructlab/dev-docs/pull/161) dev doc.

### Future phases

- In the near future, RAG might be moved to the existing <https://github.com/instructlab/rag> repository.
  - If so, something will be done with the existing code in <https://github.com/instructlab/rag>, e.g., moving it to a branch of that repository or moving it to a different repository.
- Alternatively, some or all of it might move to a new repository.
  - For example, maybe the indexing and retrieval portions move to a separate retrieval repository while the rest of end-to-end runtime RAG might move somewhere else.
- If/when we move ahead with any of these options, *we will open a new ADR for that decision*.
- Also, the capabilities will keep improving and adding more functionality.

## Alternatives

- Put the indexing and run-time RAG code in a new repository.
  - Pro: Having a dedicated repository gives the RAG team the most freedom and flexibility to make technical decisions that work for that team.
  - Pro: Starting with a new repository provides a blank slate that can be set up in whatever way makes the most sense for that functionality.
  - Pro: Having the capability in one repository makes it easier for consumers such as RamaLama to reuse it for their purposes too.
  - Con: Creating and configuring a new repository is some work.  (This is a fairly small con, but a real one.)
  - Con: Integrating a new repository into the continuous integration and delivery capabilities for both upstream InstructLab and downstream consumers is a *lot* of work.  This is a much bigger con.
  - Con: All that extra work would almost certainly result in slower time to market.  This risks missing some market opportunities.
- Put the indexing code in <https://github.com/instructlab/sdg> (SDG) and the run-time RAG code in <https://github.com/instructlab/instructlab> (core)
  - Pro: This has the advantage of not adding any new dependencies.
  - Pro: The document processing is already in SDG and chat functionality is already in core so this would require the fewest code changes.
  - Con: Splitting the RAG functionality across multiple repositories makes it more complicated to reuse in other applications outside of InstructLab.
  - Con: Many things we will want to do to add advanced functionality to make RAG more effective will require changes to both indexing and run-time RAG.  If those components are split across multiple repositories, that will make delivering such changes more complicated.
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
  - Con: It creates the confusion of having two different RAG solutions in the same repository.  We could mitigate that with developer documentation and marking legacy stuff as "deprecated".
- Put the indexing and run-time RAG code in <https://github.com/instructlab/rag> AND eliminate the existing RAG functionality in that repository.
  - Pro: It already exists.
  - Pro: It avoids the confusion of having two different RAG repositories in <https://github.com/instructlab/>.
  - Pro: It avoids the confusion of having two different RAG solutions since we'd be eliminating the old one.
  - Con: There is still some interest in keeping this around.

## Risks

- Putting the RAG functionality in the core repository requires any application that wants to use this functionality to bring in the entire core which then brings in all of the libraries it depend on, so this becomes an enormous dependency.  This discourages reuse in other applications.  It *encourages* either of the following behaviors that would be unfortunate:
  - Other applications pull directly from <https://github.com/redhat-et/PaRAGon> and in doing so duplicate the ongoing effort to harden that code base.
  - Other applications may implement their own RAG solutions or pull from some other upstream unrelated to ours.
- As noted earlier, putting the capability inside <https://github.com/instructlab/> signals that this is a component of InstructLab and not a generally useful feature.  That creates a risk that the work could miss out on additional opportunities for impact.  We hope to mitigate that risk by spinning it off to its own open source project when it is mature enough, but there is a risk that we will get distracted by other things and never get around to this.
- The flow for document processing for InstructLab winds up being quite complicated in this proposal.  Since the existing document processing is in SDG, the flow for indexing for RAG winds up being a bit complicated (i.e., it starts with a CLI call handled by the core repo then goes to SDG for some of the document processing and then back to the core `/data` directory which then calls out the the `core/rag` directory for chunking and vector database indexing).  Having the document processing move from core to SDG and back to core and forward to RAG makes that capability more difficult to understand and maintain.  This complexity will be partially mitigated when the preprocessing code moves from SDG to core.  It will be further mitigated by having a clear, well-documented contract between core and the RAG repository indicating the responsibilities of each.

## References

- <https://github.com/redhat-et/PaRAGon>
- <https://github.com/instructlab>
- <https://github.com/instructlab/rag>
