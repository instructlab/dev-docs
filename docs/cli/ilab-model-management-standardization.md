# Standardizing model storage, management and referencing in the CLI and libraries

## Introduction

- **Purpose**: Standardizing how models are stored, managed and accessed via flags used in CLI tools and libraries to ensure consistency, usability, and maintainability.
- **Scope**: Covers model directory structure in cache and all model-related flags used in the command-line interface (CLI) and libraries within the project.
- **Audience**: Developers, maintainers, and contributors involved in the development and usage of the CLI tools and libraries.

## Summary

This proposal aims to establish a consistent practice for how models are managed within InstructLab. It does so in 2 parts - one proposing a consolidation of all existing model related flags, and the other proposing a change to how models are organized in the cache upon download, and thereby referenced within various operations

## Motivation

Currently there are 10+ instances across all major commands and subcommands that accept either `--model` , `--model-path`, `--model-name`, or `--model-dir`; all of which serve slightly different purposes, and might handle different use cases including local vs remote models, relative vs absolute paths etc. This leads to a significant amount of avoidable confusion among developers and users alike. Introducing some consolidation and consistency in this aspect would provide a big UX improvement.

In addition to this, we currently support multiple sources for model downloads. There could be models that span multiple sources and also have different versions and tags that users may want to be able to work with. Currently, there is a lack of uniformity in how we organize models downloaded from Hugging Face vs OCI registries. We do not have a way to differentiate and maintain unique identities for models in cache, based on their sources. We also currently don't have the ability to support version management. Users are only able to work with one version of a given model at a time, as downloading a different version just overwrites their existing model.

## Proposal

### Part 1: Narrowing down to a singular model related flag

#### Problem Statement - Section 1

We want to streamline the number of model related flags available, while establishing consistency and predictability between their uses and handling 3 separate use-cases: absolute paths, relative paths and remote repository names.

#### Suggested approach - Section 1

- Retain `--model` and deprecate all other model related flags
  - Alternatively, we could retain `--model-path` but passing in a repo name to a flag called "model-path" seems a little more awkward than passing in a path to a field just called "model"
- `--model` should accept both paths, as well as strings (for repo names)
- First `--model` should simply check whether the supplied path exists - This would include determining whether the provided path is absolute, or relative to wherever the calling program is.
  - If it exists, we should run an additional check to determine if it points to a safetensor or gguf models (use existing `is_model_safetensors` and `is_model_gguf` checks for this)
  - If path does not exist, or exists but is not determined to be a valid model, then move on
- If path is not found after step 1, check against all tracked models/checkpoints/adapters etc currently found in `~/.local/cache` and or `~/.local/checkpoints`. If not, move on
- If supplied content is neither an absolute path, nor a relative path - assume it is the name of a remote repo on HF and download it
  - alternatively, we can error out here and require that user download the model explicitly via `ilab model download`

This would standardize the behavior of the `--model` flag across all the commands that it appears in. There could be a dedicated model resolver function that implements the above described process.

The only exception may be `ilab model download` which contains a `--model-dir` flag, which acts as a sink rather than a source. This flag could stand to benefit from being renamed to `--destination` instead.

### Part 2: Standardizing what gets passed _into_ the model flag (value)

#### Problem Statement - Section 2

In addition to standardizing the flag itself, we must also standardize what gets passed INTO the flag, i.e the value passed to `--model`. The format used to reference models should work consistently regardless of whether the user is referencing
a local model or a remote one. They should be able to use a consistent string to reference a given, specific model under all circumstances.

#### Suggested approach - Section 2

- Standardize around the usage of a model's full URL as the way to reference that model at all times (e.g `quay.io/ai-lab/models/granite-7b-lab`).
- Download logic is updated such that models are always downloaded into `~/.local/cache/instructlab/models` under sub-directories that follow the same structure as their URL. We also account for model versioning by doing the following:
  - Creation of dedicated sub-directories based on tag/branch/commit SHA
  - Creation of a `.metadata` file that records the tag/branch/commit information
- Keeping these in mind, an example for what the final model path could look like is: `~/.local/cache/instructlab/models/quay.io/ai-lab/models/granite-7b-lab/v1.1`
- Users use the full URL `quay.io/ai-lab/models/granite-7b-lab` when specifying where to download the model from. Thereafter, users continue to use this string (with the version included) to reference this model from cache, as this will now match the relative path of the model (based on point #4 of suggested approach of part 1)
  - E.g: `ilab model chat --model quay.io/ai-lab/models/granite-7b-lab/v1.1`
  - Alternatively, we could adopt standard convention and allow specification of version through a colon (`ilab model chat --model quay.io/ai-lab/models/granite-7b-lab:v1.1`) and extract the version from this URL and use it to locate the right version folder
  - There could be some UX enhancements we add on top, such as allowing users to just specify "granite-7b-lab" and use some logic to determine which sub-folder and version to default to if multiple copies of that model exist across different sources and versions, similar to Podman

An ongoing work-in-progress effort for this can be found at: [#1895](https://github.com/instructlab/instructlab/pull/1895).

The proposed combined approach will resolve [#2200](https://github.com/instructlab/instructlab/issues/2200), [#1871](https://github.com/instructlab/instructlab/issues/1871) and all issues associated with it.

#### Open question

One issue with following the suggested approach would be that we might like to store Hugging Face models under `huggingface.co/` in the cache, similar to `quay.io/` for example. However, the Hugging Face API expects users to specify models following the pattern
`<repo-name>/<model-name>` and does not accept `huggingface.co/<repo-name>/<model-name>`. As such, we could still store the models that way and have users specify `huggingface.co` for the sake of uniformity and have some logic to strip out the `huggingface.co` from the URL before sending the API request. This seems rather clunky and unnecessary. On the other hand, if we continue storing Hugging Face models in the same `<repo-name>/<model-name>` format, it would break the pattern with most models being
collected under their host domains, and Hugging Face models arbitrarily stored one level higher than the rest. What's an acceptable solution in this case?

## Follow up work

- `ilab model list` is updated to include a `version` column that displays all available versions for a given model. This logic should read from the `.metadata` files to pull the version info. Existing models already downloaded by models won't contain this file and hence should automatically have a version of `n/a` since we cannot determine the version of their models after the fact
- `ilab model list` is updated to accept `--adapters` and `--checkpoints` flags to act as filters and contain dedicated sections to list model adapters and checkpoints
  - The existing `--list-checkpoints` could be deprecated for uniformity reasons

## How would backwards-compatibility be handled?

All other model flags will be deprecated for a couple releases and called out in the release notes. They will eventually be removed.
The fields in the config file will need to be updated to match `--model`, which might be a breaking change and may warrant bumping the config version. This might require implementation of some kind of automatic config conversion mechanism between versions