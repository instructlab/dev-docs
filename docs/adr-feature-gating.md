# InstructLab Feature Gating Patterns

## Context

Motivated by the currently [in-progress RAG development](https://github.com/instructlab/dev-docs/pull/161) in [dev preview scope](https://access.redhat.com/support/offerings/devpreview), there is a use case for introducing feature gating (sometimes equivalently called "feature flags"). The specific use case desired is to have default settings for feature gating in the application that can be overridden using environment variables.

One common pattern is per-feature gating, i.e., configuration per-feature. This fine-grained control might be desirable in the future, especially to enable user segmentation for individual experimental features. We do not need that level of fine-grained control at this time.

There is precedent [in OpenShift](https://docs.openshift.com/container-platform/4.17/nodes/clusters/nodes-cluster-enabling-features.html) for enabling sets of features based on support scope. Following this pattern would be consistent with OpenShift terminology and meets our needs at this time.

## Decision

InstructLab will adopt feature gating based on feature sets using the OpenShift terminology of `DevPreviewNoUpgrade` and `TechPreviewNoUpgrade`, able to be overridden using an environment variable.

## Status

Proposed

## Consequences

* Feature gating concepts will be consistent with OpenShift, lowering the learning curve of one application's configuration when coming from the other.
* We will not need to (*yet*) spend the time to develop our own taxonomy for feature gating.
* Messaging to users about support commitments when using dev preview or tech preview will be clear - in particular, that no version upgrade commitments are made, nor the ability to disable those scopes in order to revert to a supported application state.
* We have a decision to make about whether to introduce a new dependency for feature flagging or make our own simple one.
* There is a migration path to finer-grained feature gating via the [`CustomNoUpgrade`](https://github.com/openshift/api/blob/master/config/v1/types_feature.go#L54) scope.
* We will have to make sure to communicate what these feature gate scopes mean to users and what commitments they entail, in documentation and/or otherwise.
