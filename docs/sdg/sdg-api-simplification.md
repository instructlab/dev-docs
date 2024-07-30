# SDG API Simplification

## Objective

Identify simplifications to [the original SDG API design](sdg-api-interface.md) based on retrospective insights from working with the implementation of that design.

## Original API Design

Consider the original API sketch:

```python
from sdg import SDG
from run_config import SynthDataFlow
from pipeline import Pipeline
import yaml

client = openai_client(endpoint)
model = "model-version"

synth_skills_flow = SynthDataFlow(client, model).get_flow()
skills_pipe = Pipeline(synth_skills_flow)

cli_sdg = SDG([synth_skills_flow])  # run config has all the variables like num_samples, pipelinesteps etc
generated_samples = cli_sdg.generate()
```

The nouns above are:

* Dataset - this is from Hugging Face's datasets library - used for the return value from `SDG.generate()`, but also what is passed between elements of the data generation pipeline
* Block - not shown in the code above, but required to understand a pipeline - a block provides a `generate()` method transforms an input dataset and returns an output dataset
* Block config - a description of how to instantiate and invoke a block, a sequence of these is returned from `get_flow()` above
* Flow - a class which describes how to render a sequence of block configs for a pipeline
* Pipeline - a pipeline is created from a sequence of block configs, and provides a generate() method in which it instantiates and invokes blocks in turn, passing the input dataset and collecting the output
* SDG - an SDG is created from a list of pipelines, and its generate() method calls pipelines in turn

## Simplification Proposals

### Remove `SDG`

We don't need both SDG and Pipeline since Pipeline can already do everything SDG can do. If more advanced orchestration of multiple pipelines is required later, an orchestration abstraction can be added then.

### Remove `Flow`

With flows migrating to a YAML file format (#109), their purpose becomes more clear - they are simply an expression of the configuration of a sequence of blocks, used to create a pipeline. We can simply refer to these YAML files as pipeline descriptions.

### Add PipelineContext

A number of runtime parameters are required by blocks in a pipeline - e.g. every `LLMBlock` requires an OpenAI API client and a model name. These parameters are distinct from configuration that is specified by a pipeline author.

It would be much more straightforward if `Block` were able to access these runtime parameters via their parent `Pipeline`.

In the case where multiple pipelines with the same runtime context is desired, it would also be beneficial to abstract these runtime parameters into a `PipelineContext` class.

## New API Design

```python
ds = Dataset.from_list(samples)

ctx = PipelineContext(client, "mixtral", teacher_model)

knowledge_pipe = Pipeline.from_configs(ctx, [MMLU_BENCH_PIPELINE, SYNTH_KNOWLEDGE_PIPELINE])

gen_data = knowledge_pipe.generate(ds)
```
