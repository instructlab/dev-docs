# InstructLab Chat Module Architecture

## Context

InstructLab recently gained the ability of [retrieval augmented generation (RAG) for chat](https://github.com/instructlab/instructlab/pull/2886). The initial implementation is the simplest possible, inserting retrieved document chunks in the chat session history before submitting the user query to the target model. This is done by adding a retriever instance into the chat class which is called by submitting the user query as-is to the vector store as a search query to retrieve those chunks.

Tuning retrieval systems requires significant expertise and testing, and there are many techniques that may be needed specifically and only for that use case. For example, query reformulation or agentic approaches may be necessary to achieve reliable retrieval quality while being redundant and expensive in a non-retrieval scenario.

Having a single code path that conditionally executes various strategies based on a mixture of configuration, application state, and user input can quickly become difficult to maintain, debug, and even understand. That chat module in InstructLab would benefit from a principled approach; that is, to be purposefully architected before becoming a [big ball of mud](http://www.laputan.org/mud/?ref=blog.codinghorror.com).

Prompt template management, model interaction pattern, chat session history, and any interactions with third party systems can vary by use case and so should be logically grouped for purposes of at least maintainability, understandability, modifiability, and testability.

## Decision

The InstructLab chat module will adopt a [strategy pattern](https://en.wikipedia.org/wiki/Strategy_pattern).

## Status

Accepted

## Consequences

* A refactor to the chat module will be necessary before additional feature work.
* Encapsulation of different sets of logic as distinct pattern should decrease the risk of regression in one area when implementing another.
* Division of responsibility will be clear from the code and project structure.
* Testing different chat strategies will become simpler, with a smaller total testing surface.
* Development velocity of RAG-specific improvements should be higher.
* Code review will become simpler.
* Risk of unsustainable maintenance overhead should be decreased.
* As blocks of logic common to multiple strategies emerge over time, there will be a natural path to a pipeline approach in the future.
* Configuration might become more complex; care should be taken in designing it to be flexible and to avoid one-way doors.
