# Taxonomy Caching Service (TCS)

This document describes concept of a  service that is intended to preserve Taxonomy knowledge documents as they were at the time they were merged into the Taxonomy repository

## Motivation for this service

Taxonomy knowledge is submitted and reviewed at a point in time, we need to protect against the content changing(if/when URLs are supported) or becoming unavailable either temporarily or permanently.

At any point in time we (or a 3rd party) should be able to view the knowledge which was used to train models. Not only should we be able to access the content representing the current HEAD of the taxonomy repository but we should also be able to look back at the content as it was at any point in time.

In order to fulfill these requirements we need to keep a highly available cache available for manual browsing and for training backends to reference.

This service's primary objective is to preserve knowledge documents' integrity, ensure availability, and  support training backends.

## Overview

### Components

#### Caching service

The caching service is responsible for populating the cache with relevant knowledge documents, it should run local to the cache and will populate the file system directly.

#### Cache

The cache is the store of knowledge, served over https and publicly available. It contains all knowledge documents currently and previously referenced in the taxonomy repository.

### Knowledge Versioning

Each version of each taxonomy document (qna.yaml) referencing external knowledge in the taxonomy repository will be given a unique URL. This will contain an index of knowledge documents referenced by it, these knowledge documents will be stored alongside the index.

This URL will take the format
$BASEURL/$SHARDEDGITHASH/$TAXONOMYPATH

where
BASEURL: Is the base URL leading up to the root of the cache
SHARDEDGITHASH: A sharded version of the hash in which the qna.yaml file was updated in the taxonomy repository
TAXONOMYPATH: The full path to the qna.yaml document in the taxonomy repository

e.g. for the taxonomy file knowledge/technical_manual/instructlab/qna.yaml
the unique URL would be
https://tcs.instructlab.com/5/5/3/111a7e333bdd6d885c568e2b628ad09f0485de/knowledge/technical_manual/instructlab/qna.yaml

This YAML file will then contain the document section form the taxonomy qna.yaml file, along with a list of files that were cached e.g.

```
document:
  repo: https://github.com/instructlab/instructlab/
  commit: cc35593f4e4e348dfba9c2d84b06a46bc84ec42e
  patterns:
    - *.md
files:
  - path: README.md
    cachefile: 1.yaml
    sha256: fbcfc375f60...f254
  - path: CODE_OF_CONDUCT.md
    cachefile: 2.yaml
    sha256: fbcfc375f60...f254
status: ok
```

The cache files stored alongside the index do not maintain the same filename as their original source in order to prevent filename collisions that may occur e.g. when knowledge files have the same name in a different directory.

### Format Stored

The content stored should be the exact file as retrieved. Any conversions and processing that are required for training should be done after retrieval  of these documents from the cache.

### Caching Service Trigger

The caching service can be triggered either immediately when a new commit is merged into the taxonomy repository or periodically. Regardless of how it is triggered it should
* run locally to the cache
* Gather a list of all commits from where the last run left off(only following the first parent of each commit)
* create a new cache entry for each commit that updates a qna.yaml file containing knowledge

### Reading the taxonomy

Training tools and backends should refer to this cache as the single source of truth for knowledge documents. Initially this will guard against git repositories going missing. But in future if arbitrary URLs are supported for knowledge documents then this will guard against the content of these URLs changing after initially cached.

### Cache storage

As the exact capacity and performance requirements of the cache are as yet unknown, care should be taken so that the chosen solution can be expanded as required.

The sharding of references described above will allow the cache to be more easily spread between storage devices or servers as required in future. It should also help in preventing individual directories from getting overly populated.

### Access

Public read only access to the cache should be provided over https.

### Implementation

The implementation will be discussed once this document is agreed upon but two potential options could be to implement a `ilab cache` command or create a new stand alone project.

## Risks

### Knowledge becomes unavailable between review and caching

There continues to be a risk that a git repository would become unavailable between being reviewed and getting cached. Keeping this window to a minimum will reduce this risk.

The caching service should log and alert when this occurs so that problems can be manually dealt with. A blank index will be stored in the cache with a status set to “incomplete” to mark such cases.

In future documents could be captured into a temporary cache when a PR is submitted. From here they could be reviewed. Once merged the permanent cache could be populated from the temporary cache.

## Alternatives

### Do nothing

The risk of doing nothing will result in the quality of the taxonomy repository deteriorating over time as knowledge documents can no longer be retrieved. Or URLs contain content that was never reviewed.

### Git repository

All of the knowledge could be stored in a taxonomy 'sister' repository that gets updated with knowledge as references to new documents are added to the primary repository.
This could fulfill the requirements but may come with a risk that the size of the git repository would eventually make it practically unusable.

## Out Of Scope / Future Work

### PR reviewing cache

In future a temporary cache can be built that would be populated when knowledge PR's are submitted. The permanent cache would then be populated from this when the PR is approved

#### knowledge diffs

To aid reviewers in reviewing updated knowledge documents, any PR refreshing the contents of already existing knowledge documents could be provided with a diff between the permanent cache and the temporary cache.

### Cache Mirroring

There should be no need to duplicate the caching service but the cache itself could in future be mirrored to provide high availability to services reading it. If required the data could be pushed to a suitable CDN.

### knowledge URLs

In future knowledge documents may be referenced by URL (in addition the the git references currently supported), this is out of scope for the initial version of the TCS but may be added in future.
