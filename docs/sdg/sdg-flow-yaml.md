# SDG API - Add a file format for defining custom Flows

## Problem Statement

The `instructlab/sdg` library is introducing more extensive data generation pipelines. To enable customization, we should allow users of the library to provide a configuration file which defines a custom pipeline or extends an existing pipeline.

In terms of the API constructs, a Pipeline is created from a sequence of “block configurations” which express how to instantiate and invoke the individual steps (aka blocks) in the pipeline. A Flow construct serves as a template from which a sequence of block configs can be generated.

## Objective

- Library users can specify a custom flow using a well-defined file format.
- Library users can either use a custom flow standalone, or combine a custom flow with existing flows.
- The file format and library can evolve substantially without making breaking changes.
- Incompatible changes can be introduced while retaining support for existing custom flows for a deprecation period.

## Proposal

### Existing API Review

The current `Pipeline` API allows instantiation with a list of `Block` configurations.
These configurations could come from one or many sources. In its simplest form:

```python
pipeline = Pipeline(block_configs)
```

or if you had two separate lists of block configurations to append together:

```python
pipeline = Pipeline(block_configs1 + block_configs2)
```

### API Additions

We will add an API that instantiates a `Pipeline` object from a YAML file:

```python
pipeline = Pipeline.from_file(ctx, 'mycustomflow.yaml')
```

The YAML file format will mirror the API and look like this:

```yaml
version: 1.0
blocks:
  - name: gen_knowledge
    type: LLMBlock
    config: # LLMBlock constructor kwargs
      output_cols: ["question", "response"]
    gen_kwargs: # kwargs for block.generate()
      max_tokens": 2048,
    drop_duplicates: ["question"]
  - name: filter_faithfulness
    type: FilterByValueBlock
    config:
      filter_column: judgment
      filter_value: YES
      operation: eq
      drop_columns: ["judgment", "explanation"]
```

## Versioning

A mandatory `version` field in the YAML file expresses major and minor versions (e.g., 1.0, 1.1, 2.0).

Compatibility rules

1. If the major version of the YAML file is higher than the parser can handle, the parser should reject the file.
2. If the minor version of the YAML file is higher than the highest version the parser is aware of, the parser should read the file but ignore any unrecognized content.
3. If the file’s version is lower than the parser version, the parser should provide default values for any configuration introduced in later versions.

Example parsing logic:

```python
def parse_custom_flow(content):
    version = content['version']
    major, minor = map(int, version.split('.'))

    if major > PARSER_MAJOR:
        raise IncompatibleVersionError("The custom flow file format is from a future major version.")
    elif major <= PARSER_MAJOR and minor > PARSER_MINOR:
        logger.warning("The custom flow file may have new features that will be ignored.")
```

### Pipeline Context

The following runtime parameters will no longer be part of the pipeline configuration definition and instead available to blocks via a `PipelineContext` object:

- client - an OpenAI completions API client for talking to the teacher model via the serving backend (i.e. llama-cpp or vLLM)
- model_family - e.g. mixtral or merlinite
- model_id - a path name for the specific teacher model being used
- num_instructions_to_generate - how many samples to generate

For now, we assume there is no need to do any sort of templating in the custom pipelines based on these runtime parameters.

### Model Prompts

Based on whether model_family is mixtral or merlinite, a different prompt is used with the teacher model

```python
_MODEL_PROMPT_MIXTRAL = "<s> [INST] {prompt} [/INST]"
_MODEL_PROMPT_MERLINITE = "'<|system|>\nYou are an AI language model developed by IBM Research. You are a cautious assistant. You carefully follow instructions. You are helpful and harmless and you follow ethical guidelines and promote positive behavior.\n<|user|>\n{prompt}\n<|assistant|>\n'"
```

For now, we assume that the `LLMBlock` class will choose the appropriate model prompt based on the family and that there is no need to specify a custom prompt.

### Prompt Config Files

Every LLMBlock references a separate prompt config file, and presumably a custom pipeline will provide custom prompt configs too.

These prompt config files are quite simple YAML files - they contain a single object with system, introduction, principles, examples, and generation keys. See e.g. src/instructlab/sdg/configs/skills/freeform_questions.yaml

We will continue to use these config files unchanged, and custom files can be specified with an absolute path. Relative paths are assumed to be relative to the Python package e.g. `configs/skills/...`.

### Model Serving

Custom pipelines may have more unique model serving requirements. Instead of serving just one model, we may need to launch the model server with a model and an additional model with adapter. vLLM, for example, can host both a model and a model+adapter under two different model IDs.

The pipeline author needs some way of disambiguating between these multiple models - i.e. the definition of each `LLMBlock` needs to specify a particular model.

Right now the `Pipeline` constructor takes two relevant parameters - the OpenAI client instance, and the model ID for the default model. It's important to note that this model ID is defined by the user at runtime, and it may not match the model IDs that the pipeline author used.

The use cases will be:

1. Most LLMBlock definitions will use the default teacher model - and we can make the semantic that if the pipeline author doesn't specify a model in an `LLMBlock`, the default in `PipelineContext.model_id` is used.
2. In cases where a model+adapter is to be served, the pipeline author should choose a descriptive model ID using `block.gen_kwargs.model_id` and the user should ensure that this is the model ID that is served.

For example, a pipeline author might define:

```yaml
version: "1.0"
blocks:
  - name: gen_questions
    type: LLMBlock
    config:
      config_path: configs/skills/freeform_questions.yaml
      add_num_samples: True
      gen_kwargs:
        model_id: mycustomadapter
      output_cols:
        - question
    drop_duplicates:
      - question
```

and the user will be required to define a serving configuration like:

```bash
--lora-modules=mycustomadapter=path/to/my_custom_adapter
```

### Re-use of Built-in Pipelines

A custom pipeline may want to extend an existing built-in pipeline. In that
case, a new block type, `ImportBlock`, may be used to import the blocks from
another configuration file.

```yaml
version: "1.0"
blocks:
  - name: import_from_full
    type: ImportBlock
    path: configs/full/synth_freeform_skills.yaml
  - name: custom_post_processing_block
    type: LLMBlock
    ...
```

### CLI Integration

As of the current version of `ilab`, it supports `simple` and `full` as parameters to `--pipeline` to select one of the two types of built-in pipelines included in the library.

Once we have support for loading custom pipelines, we need a way for these to be specified with the CLI. We believe the most common case for custom pipelines is for them to extend the `full` pipeline and, as such, we should support extending existing pipelines with a custom pipeline rather than simply specifiying a single pipeline.

Here is a proposed CLI UX for this:

> `ilab data generate`

Use the default pipeline, `simple`.

> `ilab data generate --pipeline full`

Use the built-in `full` pipeline.

> `ilab data generate --pipeline path/to/custom_pipeline_directory/`

Use a custom pipeline configuration. The custom pipeline may include references to the built-in flows to be used in conjunction with custom ones, but those details are contained within the yaml files in the custom directory.

### File and Directory Structure

The existing contents of `default_flows.py` will become these files in the source tree:

```text
src/
  instructlab/
    sdg/
      pipelines/
        simple/
          knowledge.yaml
          freeform_skills.yaml
          grounded_skills.yaml
        full/
          knowledge.yaml   # also contains the current contents of mmlu_bench.yaml
          freeform_skills.yaml
          grounded_skills.yaml
```

When the `--pipeline` option to `ilab data generate` is used to point to a
custom directory, we will assume that the same 3 files are present. All three
files will be loaded and used according to the type of taxonomy additions
present when running `ilab data generate`.

### Future CLI Improvements

A possible improvement would be to have a well-defined place on the filesystem where custom pipeline configs can be automatically loaded and included as options to the `--pipeline` parameter.

For example, if the config format included new parameters, `name: full-extended` and `extends: full`, and the CLI discovered and loaded it automatically, we could support `--pipeline full-extended` without needing the additional `--pipeline-extend` option.

`/usr/share/instructlab/sdg/` is a proposed location for this as a place for a distribution of InstructLab to include pre-defined custom pipelines, at least for Linux. See the [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s11.html) for more details on why this path is appropriate for this use-case.

It would also make sense to support a configuration directory for user's own custom pipeline configurations. Assuming there is a base config directory, these could go in a `sdg` subdirectory. There is a separate proposal that discusses a proposed configuration location: <https://github.com/instructlab/dev-docs/pull/104>. Note this is separate from the distribution-provided, read-only pipelines discussed above with a different location.

If we have a location with pipeline examples then a nice to have would be to have a `ilab data generate --list-pipelines`.

## Alternative Approaches

Alternatives already considered and discarded are listed below.

### No Custom Flows

It would be preferable to not support custom flows, especially so early in the project because:

- We will need an extensive API to support this customization, and we will need to be careful about making incompatible changes to that API once it has been adopted.
- We would learn more about the pipelines that users are creating if they were added to the library.

This approach was discarded because of strong demand from downstream users to define custom flows to encapsulate proprietary pipeline configuration..

### Custom Flows as Code

If we have an API for creating flows, users could define these custom flows in Python rather than with a configuration file format.

This approach was discarded because of a desire by downstream users to separate reusable logic from proprietary pipeline configuration.

The initial version of the initial SDG library API design (#98) proposed using YAML files and this was changed to Python code based on this feedback:

> Does this need to be a yaml file?
>
> or is it actually a Python dict passed to the library?
>
> I actually think it would be a nice simplification to not worry about config files at all, and from the library perspective, assume configuration is passed in via data structures.
>
> How that config is constructed could be a problem of the library consumer. Maybe they hardcode it. maybe they allow a subset to be considered. Some could be driven by CLI args, for example.

Since adopting YAML may now appear contradictory to that feedback, it is useful to understand how the feedback relates to this new design:

1. The feedback assumes that YAML will be used for custom pipelines, but wonders whether it would be better to implement that in the CLI instead of the library.
2. Not called out is that at the time it was unclear whether custom pipeline definitions would also need to include custom model serving configuration - if so, the model serving configuration would not belong in the SDG library. It is now better understood that no model serving configuration needs to be included in the pipeline definitions. (See above)
3. The POC implementation of this format makes it clear - in a way that wasn't clear from an API design - that using the YAML format within the library is an improvement.
