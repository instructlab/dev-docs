# Refactor preprocessing and postprocessing in SDG

## Context

The existing synthetic data generation (SDG) repository includes several related pieces of functionality:

- Traverse an InstructLab taxonomy to identify unstaged qna.yaml files to generate data from.
- From each qna.yaml file, extract the example context/question/answer tuples for use as seed data.
- From each *knowledge* qna.yaml file, also fetch the document referenced by this file.  Use [Docling](https://github.com/DS4SD/docling) to convert the file to a JSON format and then split the file into chunks that are small enough to be used as contexts for synthetic data generation.
- *Given the seed data and the document chunks if any, generate additional synthetic context/question/answer tuples.*
- Mix the outputs with some pre-computed data sets when applicable
- Split the data into train and test

Of all of these, only the one emphasized (*Given the seed data ... generate ... tuples*) is SDG functionality.  The others are essentially preprocessing and postprocessing steps to enable the SDG functionality and produce outputs usable for future steps.  In the current flow, preprocessing has a taxonomy with some new seed data added to it as input.  The output of preprocessing includes a set of context/question/answer tuples for both knowledge and skill taxonomy nodes.  For knowledge taxonomy nodes it also includes a set of document chunks.  SDG uses the context/question/answer tuples as seed examples, and it uses the document chunks (if there are any) as example contexts from which to generate additional data.  That additional data is then sent to the postprocessing step to produce the final outputs.

We have heard that some users want a stand-alone SDG capability that includes only the SDG functionality.  Specifically, they already have a set of seed context/question/answer tuples and optionally a set of document chunks.  All they want from SDG is to take that input and produce an new synthetic data set as output without doing any mixing into pre-computed data or splitting into train and test.  The preprocessing and postprocessing capabilities currently in SDG are not relevant to those users.

Also as context, in the near future we are absorbing a set of updates to the SDG functionality to make it more modularized and flexible.  That might turn out to be irrelevant to this document which is focused on what to do with the preprocessing and postprocessing.  However, it is mentioned here in the context section in case that context winds up being useful.

Furthermore, in 2025 we are hoping to have some sort of retrieval-augmented generation (RAG) capability that is either part of or tightly integrated with InstructLab.  Such a capability would have significant overlap with the functionality of the preprocessing for SDG.  As noted above, when a taxonomy has a knowledge qna.yaml file that references a document, SDG uses Docling to convert the file to JSON and then splits the file into chunks of appropriate size for SDG.  The RAG capability would *also* want the same Docling JSON output but would need to split it into chunks that are sized appropriately for vector retrieval (i.e., that fit within the context window of the semantic encoding model).

An additional complication is the fact that InstructLab's existing "taxonomy" structure is a tree structure encoded as a git repo that can be cloned/pushed/shared using the normal git constructs and flow.  A taxonomy has *staged* nodes that are presumed to already be fully addressed by the model and *unstaged* nodes that are not, which is why the first item in the list above involves identifying only the unstaged qna.yaml files.  However, some users might have the essential elements of a taxonomy (seed context/question/answer tuples for both skills and knowledge plus documents for knowledge) but do not want to put that information in a tree it a git repo.  For the purposes of this document, we will refer to those essential elements as "raw seed content".  The "raw seed content" includes all of the things that go into a qna.yaml file.  In the current code base, the way InstructLab gets to the raw seed content is by identifying unstaged qna.yaml files from a local clone of a taxonomy.  However, in the future we might add functionality that allows users to simply point at some raw seed content without having to tie it to a github repository for a taxonomy.  If the raw seed content includes knowledge elements (not just skills) then those knowledge elements will have references to documents.  When the raw seed content is processed, the documents are fetched, converted, and chunked (the third step in the list above).  For this document, we will use the term "processed seed content" to refer to the outputs of that processing.  So to summarize the data structure terms being discussed here:

- *Raw seed content* -- A set of elements each of which has a set of context/question/answer tuples.  Some elements may be *knowledge* elements which also have references to documents.
- *Processed seed content* -- The same as raw seed content except all references to documents are replaced with a set of document chunks of appropriate size for SDG.
- *Taxonomy* -- A tree structure encoded as a git repo.  Some leaves of the taxonomy are unstaged, indicating that they should be used for raw seed content.

## Question 1: What user flows should be supported?

Here are some user flows that seem like they might be valuable:

1. User installs the full InstructLab (command-line interface and/or graphical interface).  They want any of the following using command-line or graphical interactions:
    - 1.1. They have raw seed content.  They want to run the full pipeline including SDG and model training and evaluation.
    - 1.2. They have raw seed content.  They want to run SDG and then evaluate an existing model on the outputs of that SDG.
    - 1.3. They have raw seed content.  They want to run SDG only.
        - 1.3.1. They also want to see the inputs to SDG that get extracted from the raw seed content (i.e., a set of seed context/question/answer tuples and with document chunks for the knowledge if any).
        - 1.3.2. Alternatively, maybe they only want to see the inputs to SDG -- they don't actually want to run SDG.
    - 1.4. They have processed seed content.  They want to run the full pipeline including SDG and model training and evaluation.
    - 1.5. They have processed seed content.  They want to run SDG and then evaluate an existing model on the outputs of that SDG.
    - 1.6. They have processed seed content.  They want to run SDG only.
2. User installs the SDG library only.  They want to invoke any of the following as a library call from code they write:
    - 2.1. They have raw seed content.  They want to run SDG only without any postprocessing.
    - 2.2. They have processed seed content.  They want to run SDG only without any postprocessing.

If I understand the latest guidance from our product management, the flows that our users want us to support here are 1.1., 1.2., 1.3, 1.3.1, and 1.6.  In an earlier draft of this proposal, I had said that I thought product management also wanted 2.2, but the latest guidance doesn't seem consistent with that understanding.  I am still not sure, so more clarification would be helpful.

## Question 2: What should the commands be in the command-line interface?

One way to support both 1.3.1 and 1.3.2 would be to have separate commands for the preprocessing, SDG, and postprocessing step .  Alternatively, a single command that does all of these and also saves the outputs of preprocessing to disk would support 1.3.1 but *not* 1.3.2.  Even if we only want to support 1.3.1, having separate commands for each step might be desirable because it is just more intuitive that if a user wants to save the outputs of preprocessing to disk to have a command to do that instead of having it be a "side effect" of an omnibus SDG command.  Here is a rough outline of what the separate commands would be:

- There would be a command to handle all the preprocessing (the first three bullets in the Context section above, plus any additional preprocessing we add in the future).
- There would be a command that runs the synthetic data generation *only*.  If this command replaces the existing `ilab data generate`, that would be a breaking change from the current behavior of `ilab data generate`, but that may be acceptable because the user base is still small.
- There would be one or more commands to handle all the postprocessing.   This includes data mixing and arguably other postprocessing depending on exactly how one defines "data mixing".

Detailed technical specifications for these commands are outside the scope of this document and should appear in a future document instead.

## Question 3: Where should the preprocessing and postprocessing code go?

As noted earlier, currently the preprocessing and postprocessing code is in the SDG library.  Here are some options for what to do with it.

### Option 1: Leave preprocessing and postprocessing in SDG

Currently there is no documentation that I know of that explains how to do 2.1 or 2.2 (or anything else, really) with the SDG library by itself.  However, with some additional documenting and *maybe* some refactoring, it should be feasible to support both 2.1 and 2.2 in SDG.  With that said, if 2.1 is not needed and 2.2 is, then it would *also* be possible to move the preprocessing and postprocessing code out of SDG.  Some pros and cons of leaving in SDG:

Pro:

- Future changes to the input format for preprocessing and/or the output format for postprocessing (e.g., adding more expressive power to the taxonomy format) require changes to SDG *and* the preprocessing/postprocessing.  That's easier to do if they are in the same repository because they can be done in a single PR instead of multiple PRs that need to be coordinated.
- It is simpler to leave things where they are.
- If we're not totally sure which of the options we want, then it might make more sense to stick with this option for now since it avoids doing a work to move preprocessing and postprocessing *now* that could then be followed by more work to move preprocessing and postprocessing *again* after we decide where it goes.

Con:

- The logic of SDG is inherently complex and represents some of the most sophisticated and differentiating elements of InstructLab.  For that reason, it would be nice to have it in its own repository by itself.  New contributors to that core logic find it challenging enough to navigate the core functionality without having to also figure out where the core logic starts and the preprocessing and postprocessing capabilities end.  This could be mitigated by having better technical documentation (README, comments) for the SDG library.
- To the extent that the plan is for SDG to be run independently, then there will be tooling built around the SDG repo. The more tooling built around just running SDG independently the more risk of breaking contracts for that tooling. The more functionality living in SDG that isn't SDG, the more surface area there is to break.
- As noted in the Context section earlier, in the near future we are absorbing a set of updates to the core SDG functionality.  Absorbing those updates is somewhat simpler if the core SDG logic is all alone in a repository of its own.
- Keeping interconnected components in the same repository provides less pressure to consistently document the API contracts between them.  We certainly *could* have well documented API contracts for preprocessing and postprocessing and core SDG functionality that makes it clear how they interact even when both of these exist in the same repository, but it is probably more likely that we *will* do so if they are separated.
- The logic behind the core SDG algorithms are mainly developed and maintained by the Red Hat AI Innovations team (commonly referred to as the "research" team because many people on that team used to work for IBM Research) while the logic behind the preprocessing and postprocessing is mainly developed and maintained by the Red Hat AI engineering "data" team.  Having multiple teams working on a component increases the amount of coordination required.  Note, however, that preprocessing, postprocessing and core SDG all belong to the entire InstructLab community and *not* Red Hat (much less any one team in Red Hat).  So the teams really need to keep collaborating with the entire community at all times and not get into a mindset of "owning" a single piece of code.
- The expected RAG functionality in 2025 will have some complex interactions with both preprocessing and postprocessing, perhaps even involving user flows in which the core SDG functionality is not needed.  In that case, it would be confusing to have the code path for RAG include a call out to the SDG library for doing preprocessing but not actually doing the core SDG.
- It would just be simpler to explain to all stakeholders if the functionality that I've been calling "core SDG" was really just called "SDG".  We can't do that now because the SDG library has preprocessing and postprocessing in it too.

Conclusion:

- While the cons here are substantial, so are the pros.  None of the cons really seem disqualifying.  The first pro (future changes to the formats can be more self-contained) seems particularly compelling because this is a rapidly evolving field and adding new expressive power seems like something we will want frequently.

### Option 2: Move preprocessing and postprocessing into a new repository

We could have a new repository for preprocessing and postprocessing and move all the preprocessing and postprocessing code there.

Pro:

- Avoids all the cons of Option 1.
- Preprocessing and postprocessing are a coherent pieces of functionality that *could* have their own library (or libraries, FWIW).

Cons:

- Avoids all the pros of Option 1.
- Having a separate repository with its own library brings in an enormous amount of overhead in maintaining that repository (e.g., CI/CD).
- Having a separate repository with its own library also brings in an enormous amount of overhead in maintaining the core (`instructlab/instructlab`) repository's dependency on all of those libraries.
- Does not allow user flow 2.1 (because that flow includes installing *only* the SDG repository and requires preprocessing). That's OK because it is not a priority and anyway the users could approximate that flow by also installing the ingestion library.

Conclusion:

- The cost of having a separate repository is so high that we would only consider this option as a last resort.

### Option 3: Move preprocessing and postprocessing into the core repository

Pro:

- The core (`instructlab/instructlab`) repository already has a lot of "supporting" functionality. It contains most user facing logic aside from what we call the central parts of the workflow that have their own libraries (SDG, Train, Eval).  Since the preprocessing and postprocessing are non-central parts of SDG, this change would respect established precedent.  Examples of existing functionality that follow this pattern include all of the following and more:
  - download
  - serve
  - chat
  - list
  - edit
  - init
- Supporting user flow 1.3.2 requires separate commands for preprocessing and core SDG.  This is slightly simpler if preprocessing is implemented in the core repository.  If preprocessing remains in the SDG library instead then the code in the core repository would need to make separate calls to the SDG library for preprocessing and core SDG to support user flow 1.3.2.  That adds a little complexity.
- Avoids some of the cons of Option 1, but see below for some overlap.
- Avoids some of the cons of Option 2, but see below for some overlap.

Con:

- Avoids the pros of both Option 1 and Option 2.
- As with Option 1, this approach involves a lot of coordination.  There are a lot of stakeholders involved in the core repository and locating preprocessing and postprocessing there drags those stakeholders into issues relating to preprocessing and postprocessing.  However, as with Option 1, coordinating across stakeholders is something an open source project needs to do well anyway to remain engaged with the community.
- As with Option 1, this approach suffers from the fact that keeping interconnected components in the same repository provides less pressure to consistently document the API contracts between them.  In the case of Option 1, the interconnected components that would not have as much pressure to be documented would be preprocessing/postprocessing and core SDG.  In the case of Option 3, the interconnected components that would not have as much pressure to be documented would be the command-line interface code and preprocessing/postprocessing.  However, in both cases, this con could be alleviated by just having the discipline to document the APIs well even without such pressure.
- As with Option 2, this approach would not enable user flow 2.1.  That's fine since it is not on our requirements list.

Conclusion:

- This seems like a reasonable option.  The cons are mostly manageable.  However, overall the pros of Option 1 seem more compelling.

### Option 4: Preprocessing and postprocessing go to different locations

We could also mix and match any of the above options separately for preprocessing and postprocessing.  For example, preprocessing could move to the core repository and postprocessing could stay in the SDG repo.  Or preprocessing could move to a new repository and postprocessing could move to a different new repository or the same new repository.  Enumerating all possible permutations of where each could go and enumerating pros and cons of each of them would make this document unbearably long.  If anyone wants to advocate for a small number of specific permutations, we will add them to this document.

## Question 4: Should preprocessing, postprocessing, and core SDG be separate Python packages?

If we choose Option 1 (leave preprocessing and postprocessing in SDG) then we still have the option to separate them into distinct Python packages.  That would get us some of the benefits of Option 2 (moving them to a different repository) while avoiding *some* of the costs of Option 2.  In particular, it would make the boundaries clearer and put more pressure on the developers of preprocessing, postprocessing, and core SDG to have well documented contracts for how each of these elements interact.  With that said, it would also bring in some additional complexity and increase the amount of work involved.

## Decisions

- The SDG codebase will be refactored in order to modularize based on pre-processing, data generation, and post-processing steps.
- We will support the following user flows: 1.1., 1.2., 1.3, 1.3.1, 1.3.2, 2.1, and 2.1 as documented in the Question 1 section above.
- We will adopt the updates to the command-line interface that will be documented in Question 2 above.
- Pre-processing logic for SDG will be moved into the core repository as discussed in Option 3 above.
- Post-processing logic for SDG will be moved into the core repository as discussed in Option 3 above.
- The SDG codebase will be designed around the principle of "dataset in, dataset out".
- We will not separate preprocessing, postprocessing, and SDG into separate packages.

## Status

- Proposed

## Consequences

Some of the consequences are covered earlier in the pros and cons for Option 3.  Here is a brief recap of the most important of those:

- SDG preprocessing and postprocessing will join a wide variety of glue/data-format capabilities in that repository, increasing consistency.
- In the future changes to the kinds of content that SDG takes as inputs will require changes across both the SDG repository and the `instructlab/instructlab` repository.
- There will be less pressure to have a clear and well documented separation between the library APIs and the command-line interface for these functions because both are located in the same repository.  We will mitigate this consequence by being disciplined about the separation.
