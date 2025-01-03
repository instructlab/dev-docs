# Initial InstructLab Vector Store

## Context

One of the first choices to make in implementing RAG is to choose an initial vector store to develop against. Though the usage of frameworks like LangChain or Haystack make it easy to swap vector databases, we need a working end to end implementation for RAG that is tested against and available to install with InstructLab. There are many options (see [here](https://docs.haystack.deepset.ai/docs/choosing-a-document-store)).

Our main long-term requirements are that our chosen store have fully-developed document update (and thus some sort of notion of primary key), that it be scalable to cluster size, and that it have a permissive license (Apache, MIT, or similar). Among the available choices, [Milvus](https://milvus.io/) provides strategic advantage due to its [investment from watsonx](https://www.ibm.com/new/announcements/ibm-watsonx-data-vector-database-ai-ready-data-management).

Milvus can be used in-process ([Milvus Lite](https://milvus.io/docs/milvus_lite.md)), single-node ([Milvus](https://milvus.io/docs/prerequisite-docker.md)), or cluster-scale ([Milvus Distributed](https://milvus.io/docs/prerequisite-helm.md)).

## Decision

InstructLab will initially integrate with and use Milvus Lite for vector storage and retrieval augmented generation.

## Status

Accepted

## Consequences

* Users will have a clear [upgrade path](https://milvus.io/docs/upgrade_milvus_cluster-operator.md) from the laptop use case to cluster scale.
* We should be able to have access to expert resources with Milvus via IBM.
* The laptop use case of InstructLab will have a minimally resource intensive option for prototyping.
* Since Milvus is used in watsonx, we can have confidence that it can meet expected scaling requirements.
* Document updates can be accommodated using well-established [primary key functionality](https://milvus.io/docs/primary-field.md) and [partition key](https://milvus.io/docs/use-partition-key.md).
* There is a risk of developing against a mature vector store leading to usage of functionality not available in some other vector store that a potential customer requires to be used.
