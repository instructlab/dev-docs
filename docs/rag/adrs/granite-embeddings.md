# InstructLab RAG will use Granite Embeddings as the default embedding model

## Context

InstructLab RAG will be using dense vector retrieval via a vector database to select documents for use in response generation.  Dense vector retrieval requires a model to provide dense vectors of the content at indexing time and dense vectors of the query at retrieval time.

Some key considerations:

- We want a model that is reasonably effective at driving accurate semantic search.
- We want a model that runs quickly and doesn't require a lot of expensive hardware to run.
- We want a model that does not require an inordinate amount of storage space for the vectors.
- We want a model with license terms that are compatible with the license for InstructLab so that anyone using InstructLab under the terms of its license can use its default embedding model without any additional encumbrance.
- We want a model where the license terms for all of the data used to train the model are compatible with the license terms of that model.
- If possible, it would be nice for the embedding model used for RAG to be the same embedding model that is used for other purposes in InstructLab.  At this time, there are no other uses of embedding models for InstructLab, but we understand there are experiments under way to use embedding models for subset selection for synthetic data.

We don't need this to be one model that is the best possible fit for all users.  There is no such model because many of the criteria are trade-offs, e.g., models that are bigger tend to be more effective but also slower and require more memory and storage space.  Fortunately, we are just choosing a default value, and users will be free to override that default and plug in the embedding model of their choice.  For the default, the main consideration is that the model be acceptable across all the criteria.

Another consideration might be the maximum amount of text that the embedding model can produce embeddings for.  Many popular models have a maximum input length of 512 tokens, but some have longer maxima.  A larger maximum might be useful for a RAG solution in which the response generation model has a large context window.  However, it might not be: the larger context window might be better used by getting a larger number of search results or by looking up the text that came before and after each search result.  A larger maximum input length for the query might also be useful for encoding not just the current user utterance but also the history of the chat leading up to that utterance.  We will continue to explore this space and if we find that maximum input length is an important consideration, it could result in selecting a different default embedding model in the future.

### Alternatives

There are two IBM Granite English embedding models:

- [IBM Granite-Embedding-30m-English](https://huggingface.co/ibm-granite/granite-embedding-30m-english)
- [IBM Granite-Embedding-125m-English](https://huggingface.co/ibm-granite/granite-embedding-125m-english)

The 30m model provides accuracy that is comparable to 125m model on some data sets but significantly lower on other data sets. For use cases where there is a large amount of robust evaluation data and the ability to reliable metrics, it would generally be sensible to to try both and measure the speed/size/accuracy trade-offs.  On the other hand, for engagements where the quantity of data to be indexed is enormous, the advantages in indexing time and storage space from using a smaller model can be overwhelming.  Such engagements would generally be better served by 30m-English.  However, for a simple proof-of-concept where a user won't be able to measure accuracy robustly, 125m-English is probably a better choice over 30m-English because 125m is small enough for most purposes and is the one most likely to be accurate enough that a user will be happy with the results.  The simple proof-of-concept use case seems like the most important one for determining the *default* model because users who are outside of the context of a simple proof-of-concept are much more likely to be overriding the default anyway.  Both of these models have a maximum input length of 512 tokens.

Other options available include:

- IBM Granite multi-lingual embedding models seem like they could be very useful as part of a broader InstructLab multilingual strategy.  For now, the defaults we have are focused on English, but in the future we might want users to provide the target language or languages they are working with during initial setup and then defaults for various settings depend on that choice.  Since this would involve a broader end-to-end change, it is out of scope for this ADR.  The Granite multi-lingual models support 12 languages, but there are plenty of languages that they do not support.  Other models support larger numbers of languages.
- [NV-Embed-v2](https://huggingface.co/nvidia/NV-Embed-v2) has outstanding accuracy (for example, see the [MTEB leaderboard](https://huggingface.co/spaces/mteb/leaderboard) overall English and retrieval English results).  However, it is licensed for non-commercial use only and requires almost 30 gb of memory (so presumably it requires some expensive hardware to run at speed).  For non-commercial users that prioritize accuracy over every other consideration, this might still be a fine model to choose, but it does not seem like a good default value because it doesn't meet all the criteria.
- [all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) was proposed as a default in an [early draft of another dev doc in this repository](https://github.com/instructlab/dev-docs/pull/161/commits/7ca3ab624526a4c5a5c70d282f8a6be26c292020#diff-b103ed3331fbeb65d7569ea836c9fd4b53c845853d8c8e7567c34864eebcdfb8R51).  It is a very popular, lightweight embedding model.  However, its [list of training sources](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2#training-data) includes MS MARCO which is clearly marked as non-commercial use only.  In contrast, the [Granite embedding model card](https://huggingface.co/ibm-granite/granite-embedding-125m-english) asserts that "Notably, we do not use the popular MS-MARCO retrieval dataset in our training corpus due to its non-commercial license, while other open-source models train on this dataset due to its high quality."  Note that all-MiniLM-L6-v2 is licensed as Apache 2.0 which authorizes commercial use of this model, but it seems potentially problematic for the creators of a model to authorize commercial use when they trained on data that was not authorized for commercial use.  We would prefer to avoid this tricky legal concern.
- There are many other open source models of comparable size to the Granite embedding models.  However, most highly competitive models use MS-MARCO or other sources with problematic provenance.  Furthermore, IBM has [published benchmark results](https://www.ibm.com/new/announcements/ibm-granite-3-1-powerful-performance-long-context-and-more#Granite+embedding+models) showing that Granite's accuracy is highly competitive with other popular open source options of comparable size.  Note, however, that these results do not include some newer competing models.  They also don't include any much larger, more resource intensive models; such comparisons wouldn't really be *fair* (i.e., a bigger model that uses more memory has a fundamental advantage) but they would be *useful* for someone who has flexibility on hardware requirements and would like to explore trade-offs.  More benchmarking would be helpful.

## Decision

InstructLab RAG will use [IBM Granite-Embedding-125m-English](https://huggingface.co/ibm-granite/granite-embedding-125m-english) as the default embedding model.

## Consequences

- We will have consistency with the default options for model training (which are also based on IBM Granite).
- Anyone using the default option will be able to rely on IBM's outstanding, well-earned reputation for respecting intellectual property rights and careful compliance with legal restrictions.
- Anyone who wants a different balance between key considerations (hardware requirements, speed, effectiveness, terms of use, etc.) will want to override the default embedding model.  For example, users who want the best possible accuracy and have a lot of expensive hardware available to run the model will probably want to choose a larger model, and users with extremely limited hardware who can tolerate lower accuracy will probably choose a smaller model.
- We should consider strategies of offsetting perception of prioritizing IBM models over the needs of the product in addition to the clear motivation described here, such as in documentation.
- We may wish to change to a different default embedding model in the future if more benchmarking shows that a different model provides a better balance of accuracy, speed, and hardware requirements.
- More work is needed to figure out whether a larger maximum input length would be better for our RAG solution.
