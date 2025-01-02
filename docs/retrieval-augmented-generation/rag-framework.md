# RAG  will use Haystack

| Created  | Dec 4, 2024 |
| -------- | -------- |
| Authors |  Ryan Cook, Ilya Kolchinsky, Hema Veeradhi |
| Replaces | N/A |
| Replaced by | N/A |

## What

This ADR defines the decision on the framework used to support the RAG pipeline for InstructLab. The Emerging Technologies team is pushing for the adoption of Haystack to be used for the framework of the RAG offering. Haystack will handle the data ingestion and retrieval processes for the to be productized RAG solution.

NOTE: Speaking with Peter Staar on Dec 3, 2025 the Docling team is aware of the efforts to potentially use Haystack and are already looking into adding the functionality for support of Haystack.

## Why

Multiple options for frameworks currently exist. During our initial analysis, the following options were the only ones to satisfy the basic requirements in terms of functionality, reliability and open-source availability:

- [Haystack](https://haystack.deepset.ai/)
- [Langchain/Langgraph](https://www.langchain.com/)
- [LlamaIndex](https://www.llamaindex.ai/)
- [RAGFlow](https://ragflow.io/)

All of the above offer a variation of a modular pipeline architecture, where users can chain together components (like retrievers, readers, and generators) to process data in different stages.

Out of those, we propose to use Haystack for the following reasons:

1. **Focus on RAG.** Haystack is a framework specifically targeting RAG use cases and sophisticated RAG indexing and retrieval pipelines. While Langchain and LlamaIndex shine in their own areas, the former is a generalist framework and the latter has a different focus, namely building custom indices over data. Haystack provides functionality that is strongly tailored for RAG and includes a comprehensive library of out-of-the-box solutions for advanced RAG scenarios. As a result, many essential or soon-to-be-essential RAG capabilities can be implemented in a few lines in Haystack but require considerable work to be supported over Langchain or LlamaIndex. Some examples include hybrid retrieval, iterative RAG, HyDE, combining multiple ingestion sources, custom data preprocessing and metadata augmentation. As the decision discussed in this document involves only the RAG component of RHEL AI, we believe that choosing the best RAG framework, as opposed to the best general LLM serving framework, would be more strategically correct.

2. **Maturity and stability.** Haystack is the most mature, established and stable product among the considered alternatives. It has been around for more time overall (since 2017) and accumulated more mileage. Haystack has an active, sizable and steadily growing community.

3. **Extensive vendor support.** Haystack natively supports all currently popular vector DBs and provides dedicated backends for incorporating them into its pipelines. Additionally, Haystack supports multiple models and model providers out-of-the-box.

4. **Enterprise-level performance.** Haystack is designed for production-grade scalability, supporting distributed systems and high-throughput applications. Moreover, and in contrast to the alternatives (of which only LlamaIndex showcases similar performance and scalability), Haystack is specifically optimized for efficient search and retrieval in the RAG setting.

5. **Ease of use and documentation.** Being strictly focused on RAG as opposed to taking a generalist approach, the learning curve of Haystack is less steep than that of Langchain. At the same time, Haystack offers extensive documentation and tutorials which are more well-organized and easy to use than those of LlamaIndex.

6. **Architecture.** Extending the previous point, Haystack can be seen as a middle ground between Langchain and LlamaIndex, sharing their benefits while only partially inheriting their drawbacks. Like the former, Haystack enables building custom flows and pipelines. Unlike Langchain though, Haystack does not try to be too abstract and general, strictly focusing on RAG and document search instead. As a result, Haystack is more straightforward to use, especially for users looking to implement custom and highly non-standard scenarios. On the other hand, like LlamaIndex, Haystack's performance is optimized towards data retrieval and indexing, but it offers a higher degree of flexibility and better interfaces for custom use cases.

7. **Actively maintained open source project under permissive license.** Haystack is very [actively](https://github.com/deepset-ai/haystack/pulse/monthly) [maintained](https://github.com/deepset-ai/haystack/issues?q=is%3Aissue+is%3Aclosed) and [supported](https://github.com/deepset-ai/haystack/discussions). [Tagged versions](https://github.com/deepset-ai/haystack/releases) are released on a regular basis and [trusted publishing automation](https://github.com/deepset-ai/haystack/actions/workflows/pypi_release.yml) is used. Haystack is licensed under Apache 2.0, and all of its dependencies (jinja2, lazy-imports, more-itertools, networkx, numpy, openai, pandas, posthog, python-dateutil, pyyaml, requests, tenacity, tqdm, typing-extensions) are licensed under Apache, MIT, BSD or PSFL.

## Goals

- We need to identify a framework that will be used to help support the RAG work stream. This is one of many pieces involved in the RAG work stream but identifying the framework used to index and retrieve data is step 1 of the larger picture.

## Non-goals

- What can we ignore when making this decision?

## Decision

Upon acceptance of this integration our next step is to include additional ADRs for the subsequent components required for the RAG pipeline. Next up will be decisions on Milvus and containerized/non-containerized offerings of that solution.

Upon denial of this integration the team will need to go back and evaluate alternative technologies and ensure they meet the needs of the project goal and ensure they meet the larger project plan goals of a configurable RAG pipeline.

## How

A downstream should be generated of the [https://github.com/deepset-ai/haystack](https://github.com/deepset-ai/haystack) project.

## Alternatives

- Langchain/Langgraph
  - A generalist framework (as opposed to a RAG-focused solution)
  - Complicated as compared to the alternatives, steep learning curve
  - Lower performance in large-scale production environments than LlamaIndex and Haystack

- LlamaIndex
  - Limited flexibility and customization options as compared to the alternatives
  - Limited out-of-the-box support for building complex, multi-component pipelines as compared to Langchain and Haystack
  - Documentation is less well-maintained and more difficult to use as compared to the alternatives

- RAGFlow
  - Limited support for many of the mainstream vector DB providers
  - Limited scalability as compared to the alternatives

- Do not use a framework; write everything directly in Python or some other programming language
  - This would take longer to get started.
  - This would make it a lot more work to add more vector DBs since we'd need to add additional code for each vector DB we want to support. We would hide that work behind an abstraction layer the same way the frameworks do, but it is work to build and maintain the abstraction layer, and the frameworks do that for us (and have put a lot of time and effort into doing it well).
  - This would make it harder to bring in advanced functionality that the frameworks already provide. For example, Haystack provides support for RAG self-correction loops which we might want some day.
  - This might make it easier to bring in advanced functionality that the frameworks do not already provide. Frameworks provide an abstraction layer that is generally useful when you want to do things that the framework developers support but often counterproductive when you want to do things that the frameworks do not support. For example, if there is a call to the framework that collapses multiple atomic steps into a single function call, that generally makes it harder to insert your own logic in between those atomic steps.

## Risks

Future versions of Haystack can potentially introduce new dependencies, that could be:

1. Distributed under a non-permissive license (or not open source at all)
2. Not regularly and/or properly maintained

If such a situation arises, the following actions can be taken on our end:

1. Pin to the old version that doesn't have that dependency. That's often OK for a while, but eventually we're likely to run into updates that we need (e.g., critical fixes, compatibility with new vectordbs, etc.).
2. Fork the project to avoid the problematic dependencies.
3. Move off of Haystack completely.

## References

- [https://github.com/deepset-ai/haystack](https://github.com/deepset-ai/haystack)
