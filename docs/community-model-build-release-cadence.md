# Community Model Build Release Cadence

## Scope

This document is targeted to help explain the release cadence of the Community Model Build.

## Problem statement

When we build the new versions of the [granite-lab][hf_granite] model, we don't know the release cycle of "tagging" the versions. We have the repository, and there is a git system like tagging, but this doesn't set the expectations for the releases.
	At the original culmination of this project, it was understood that the community releases would happen "when they happen," but from now on, we will have an opportunity to figure out a standard process.
	We are concerned that if a user says they want to use only the "1.0.0" (major release) of the model and update it only in a "6-month cycle," for example, we have no option for this. On the other hand, a group of people may only want to run "main" (any release) and not care. This document helps determine this process.

## Proposed solution

There are two options to help resolve these use cases. First, we create a “stream” version, where someone can go to huggingface.com/instructlab/granite-7b-lab-stream and access the safe sensors whenever the Community Model Build Release team deems it necessary to release a new version.  This will resolve the issue of the “main” release cycle and ideally give the developer persona the cutting-edge changes they desire. We can build out a `gguf` release of the version and distribute this to huggingface and Ollama too.
The second option is to have a “timely” (this is open discussion at this writing) release that we tag releases on https://huggingface.co/instructlab/granite-7b-lab with specific release candidates built off of a collection of releases like this: https://github.com/instructlab/taxonomy/tree/cmb-run-2024-08-26. We can announce via the users mailing list the “timely” releases with the collections of the release branches, and if we don’t have any, we can announce that we will be skipping that versioned release.


## Next steps

Get consensus on if this is an issue from the community side (maybe we can just release and ignore the versioned releases).
If this is an issue build out granite-7b-lab-stream on huggingface.
Determine the date that the versioned release will happen.
Build a release process and documentation around both release cycles.

hf_granite: https://huggingface.co/instructlab/granite-7b-lab



