# Ingesting and Utilizing RAG Artifacts Design Proposal

**Author**: Daniele Martinoli

**Version**: 0.1

## 1. Introduction
Purpose of this document is to propose `ilab` CLI changes to extend the InstructLab flow to generate and utilize RAG artifacts
in the customer workflows. The proposal includes new commands to ingest a local RAG from pre-processed customer documentation, that we assume to be generated from docling output, and updates to the existing commands to utilize these RAG artifacts.

## 2. Use Case
* Accelerate POCs of fine-tuning LLMs + RAG.
* Generate artifacts that data scientists can use to build custom solutions in their development environment:
  * Jupyter Notebooks & Python Scripts
  * Podman Desktop
  * Red Hat OpenShift AI
  * Internal Red Hat CI systems for products or services (e.g., Lightspeed products)

## 3. Proposed Commands
**Note**: In the context of version 1.4, currently under development, no changes to the command-line interface should be
allowed. Therefore, we also propose alternative approaches to run the same RAG pipelines using existing `ilab` commands or
other provided tools.

### 3.1 RAG Ingestion Pipeline Command
The proposal is to add a `rag` subgroup under the `data` group, with an `ingest` command, like:
```
ilab data rag ingest /path/to/docs/folder
```

#### Command Purpose
This command processes embeddings generated from documents located in the */path/to/docs/folder* folder and stores them in a vector database. These embeddings are intended to be used as augmented context in a Retrieval-Augmented Generation (RAG) chat pipeline.

#### Assumptions
The documents must be in JSON format and pre-processed using the `docling` tool.

**Note**: The expected JSON schema is the same that we have when we generate knowledge data with `ilab data generate` and we
add a reference document to the `qna.yaml` document(s).

#### Supported Databases
The command supports multiple vector database types. By default, it uses a local `MilvusLite` instance stored at `./rag-output.db`.

#### Usage
The generated embeddings can later be retrieved to enrich the context for RAG-based chat pipelines.

#### Running the pipeline with v1.4
Currently, there is no `ilab data rag` command available to execute the RAG ingestion pipeline. As a result, it is not possible
to address the limitations of version 1.4 using the configuration options detailed below. 
Alternatively, we propose providing a Jupyter notebook or a standalone script to run the pipeline with default settings.

The script should guide users in overriding default options and generating the RAG artifacts in the configured vector database instance. By default, support for MilvusLite will be included.

**TODO** Introduce the evaluation framework

### 3.2 RAG Ingestion Pipeline Options
**Planned for post v1.4 (apart changes required for the `chat` command)**

| Option full name | Description | Default | CLI option |
|------------------|-------------|---------|------------|
| rag.splitter.split_by | One of `page`, `passage`, `sentence`, `word`, `line` | `word` | `--split-by` |
| rag.splitter.split_length | The maximum number of units in each split | `200` | `--split-length` |
| rag.splitter.split_overlap | The number of overlapping units for each split | `0` | `--split-overlap` |
| rag.splitter.split_threshold | The minimum number of units per split | `0` | `--split-threshold` |
| The vector DB implementation, one of: milvuslite, milvus, ... | `milvuslite` | `--vectordb-type` |
| rag.vectordb.type | The vector DB implementation, one of: milvuslite, milvus, ... | `milvuslite` | `--vectordb-type` |
| rag.vectordb.uri | The vector DB service URI | `./rag-output.db` | `--vectordb-uri` |
| rag.vectordb.token | The vector DB connection token | | `--vectordb-token` |
| rag.vectordb.username | The vector DB connection username | | `--vectordb-username` |
| rag.vectordb.password | The vector DB connection password | | `--vectordb-password` |
| rag.embedding_model.name | The embedding model name | `sentence-transformers/all-minilm-l6-v2` | `--model` |
| rag.embedding_model.token | The token to download private models | | `--model-token` |
| **TODO** evaluation framework options | | | |

Equivalent YAML document for the newly proposed options:
```yaml
rag:
  splitter:
    split_by: word
    split_length: 200
    split_overlap: 0
    split_threshold: 0
  vectordb:
    type: milvuslite
    uri: ./rag-output.db
    token: _DB_TOKEN_
    username: _DB_USERNAME_
    password: _DB_PASSWORD_
  embedding_model:
    name: sentence-transformers/all-MiniLM-L6-v2
    token: _MODEL_TOKEN_
```

### 3.3 RAG Chat Pipeline Command
The proposal is to add a `rag` option under the `model chat` command, like:
```
ilab model chat --rag
```
#### Command Purpose
This command enhances the existing `ilab model chat` functionality by integrating contextual information retrieved from user-provided documents, enriching the conversational experience with relevant insights.

#### Revised chat pipeline
* Start with the user's input, `user_query`.
* (optional) Refine the `user_query` using the configured LLM to generate a semantically enriched `context_query`.
* Use the `context_query` (or the `user_query`, if the previous step was disabled) to retrieve relevant contextual information from the 
  embedding database.
* Append the retrieved context to the original LLM request.
* Send the augmented request to the LLM and return the response to the user.

#### Refining the user query for semantic search purposes
To improve semantic search on the embeddings database, the user query can be refined using the same or a different LLM. 
This step reduces ambiguity, filters irrelevant details, and aligns the query more closely with the database’s semantic structure, 
increasing retrieval accuracy.

To minimize LLM query costs, this refinement can be configured as an optional step in the chat pipeline.

Another strategy to reduce costs could be to perform the refinement query only when the embedding retrieval step fails to return 
sufficient contextual information to meet a configured score threshold.

#### Running the pipeline with v1.4
To address the limitations of version 1.4, we propose introducing a configuration option to enable the RAG pipeline without any changes to the CLI:
```
chat.rag.enabled=true
```
This approach allows the chat pipeline to utilize the desired RAG artifacts by relying on configuration options and maintaining
compatibility with the existing CLI. Consequently, corresponding CLI options (e.g.,` --retriever-top-k` and similar) will not be
introduced in this version.

### 3.4 RAG Chat Commands
The `/r` command may be added to the `ilab model chat` command to dynamically toggle the execution of the RAG pipeline.

The current status could be displayed with an additional marker on the chat status bar, as in:
```console
>>> /h                                                                                                              [RAG][S][default]
╭───────────────────────────────────────────────────────────── system ──────────────────────────────────────────────────────────────╮
│ Help / TL;DR                                                                                                                      │
│                                                                                                                                   │
│  • /q: quit                                                                                                                       │
│  • /h: show help                                                                                                                  │
│  • /a assistant: amend assistant (i.e., model)                                                                                    │
│  • /c context: change context (available contexts: default, cli_helper)                                                           │
│  • /lc: list contexts                                                                                                             │
│  • /m: toggle multiline (for the next session only)                                                                               │
│  • /M: toggle multiline                                                                                                           │
│  • /n: new session                                                                                                                │
│  • /N: new session (ignoring loaded)                                                                                              │
│  • /d <int>: display previous response based on input, if passed 1 then previous, if 2 then second last response and so on.       │
│  • /p <int>: previous response in plain text based on input, if passed 1 then previous, if 2 then second last response and so on. │
│  • /r: toggle the status of the RAG pipeline.                                                                                     │
│  • /md <int>: previous response in Markdown based on input, if passed 1 then previous, if 2 then second last response and so on.  │
│  • /s filepath: save current session to filepath                                                                                  │
│  • /l filepath: load filepath and start a new session                                                                             │
│  • /L filepath: load filepath (permanently) and start a new session                                                               │
│                                                                                                                                   │
│ Press Alt (or Meta) and Enter or Esc Enter to end multiline input.                                                                │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 3.5 RAG Chat Options
**Also required in v1.4**
Notes:
* All the `rag.vectordb.*` options previously defined are also used to locate the vector DB.


| Option full name | Description | Default | CLI option |
|------------------|-------------|---------|------------|
| chat.rag.enabled | Boolean flag to enable or disable the RAG pipeline | `false` | `--rag` (boolean)|
| chat.rag.context_refinement.enabled | Whether to enable the query refinement request to the LLM | `false` | `--rag-context-refinement` (boolean) |
| chat.rag.context_refinement.prompt | Prompt template for query refinement | See examples below | `--rag-context-prompt` |
| chat.rag.retriever.top_k | The maximum number of documents to retrieve | `10` | `--retriever-top-k` |
| chat.rag.retriever.min_score_threshold | The minimum score threshold for chunks to be considered | `0.5` | `--retriever-min-score-threshold` |
| chat.rag.retriever.filters | Comma separated key-value pairs filters to restrict the search space | See **Notes** | `--retriever-filters` |
| chat.rag.prompt | Prompt template for RAG-based queries | See examples below | `--rag-prompt` |

Equivalent YAML document for the newly proposed options:
```yaml
chat:
  rag:
    enabled: false
    context_refinement:
      enabled: true
      prompt: |
        You are an assistant helping refine user queries for retrieval from a vector database. 
        The goal is to extract the most relevant information for answering the user's request. 
        Rewrite this query to focus on the key concepts and details that will retrieve the most contextually 
        relevant results from the database. 
        Make it concise and specific, avoiding conversational context or ambiguity.
        Return the refined query in a single sentence. Avoid any undesired comment or consideration.
        User query:
        {{user_query}}
        Refined query:
    retriever:
      top_k: 10
      min_score_threshold: 0.5
      filters: 'url=doc.pdf'
    prompt: |
      Given the following information, answer the question.
      Context:
      {{context}}
      Question: {{question}}
      Answer:
```

**Notes**:
* Example of `filters` for a `milvuslite` DB:
```
--retriever-filters "url=https://en.wikipedia.org/wiki/Mausoleum_at_Halicarnassus"
```
which translates to the following dictionary in the retrieval request (*):
```json
{
   "operator":"==",
   "field":"url",
   "value":"https://en.wikipedia.org/wiki/Mausoleum_at_Halicarnassus"
}
```
(*) This is for `milvuslite`, other DB can have a different implementation

### 3.6 References
* [Haystack-DocumentSplitter](https://github.com/deepset-ai/haystack/blob/f0c3692cf2a86c69de8738d53af925500e8a5126/haystack/components/preprocessors/document_splitter.py#L55)
* [MilvusEmbeddingRetriever](https://github.com/milvus-io/milvus-haystack/blob/77b27de00c2f0278e28b434f4883853a959f5466/src/milvus_haystack/milvus_embedding_retriever.py#L18)


### 3.7 Workflow Visualization
<!-- https://excalidraw.com/#json=hyDopdXaChbf6TNuLGYE0,WkLQBF84vypx0JwAL5iwzA -->
Ingestion pipeline:
![ingestion-mvp](../images/ingestion-mvp.png)
Chat pipeline enriched by RAG context:
![rag-chat](../images/rag-chat.png)

### 3.8 Proposed Implementation Stack
The following technologies form the foundation of the proposed solution:

* [Haystack](https://haystack.deepset.ai/): Framework for implementing RAG pipelines and applications.
* [MilvusLite](https://milvus.io/docs/milvus_lite.md): The default vector database for efficient storage and retrieval of embeddings.
* [Docling](https://github.com/DS4SD/docling): Document processing tool. For more details, refer to William’s blog, [Docling: The missing document processing companion for generative AI](https://www.redhat.com/en/blog/docling-missing-document-processing-companion-generative-ai).
[Ragas](https://docs.ragas.io/en/latest/concepts): Framework for evaluating and optimizing retrieval-augmented generation pipelines.

## 4. Future Enhancements
### 4.1 Integrate the Knowledge Document Ingestion Pipeline
Integrate the RAG ingestion pipeline with the [Knowledge Document Ingestion Pipeline](https://github.com/instructlab/dev-docs/pull/148):
![ingestion-future](../images/ingestion-future.png)

### 4.2 Agentic RAG and Advanced Retrieval artifacts
**TODO**
...
