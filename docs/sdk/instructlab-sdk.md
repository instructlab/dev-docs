# InstructLab Python SDK

## Motivation

Today, the only way to "drive" the InstructLab opinionated workflow is via the `ilab` CLI. While this process provides a succinct way for everyday users to initialize a config, generate synthetic data, train a model, and evaluate it: the guardrails are quite limiting both in what a user can do and what the development team can add as features exposed directly to the user over time.

Additionally, current consumers of InstructLab are finding themselves importing different library private and public APIs in combination with CLI functionality to achieve the workflows they want. While these more advanced usage patterns are not for everyone, providing ways to run bespoke and piecemeal workflows in a standardized and safe way for the community is a necessity.

Unifying these various ranges of advanced workflows under an overarching `InstructLab Python SDK` will allow for new usage patterns and a clearer story on what InstructLab can and should provide as user accessible endpoints.

While each library can and _should_ have their own publicly accessible SDK, not all functionality being added to SDG, Training, and Eval needs to be correlated directly to the "InstructLab workflow". This Python SDK should, as the CLI does, expose an opinionated flow that uses functionality from the various libraries. The InstructLab SDK should be derived from the library APIs, not the other way around. SDG, for example, currently has a `generate_data` method, meant to only be accessed by InstructLab. This method simply calls other publicly available SDG functionality. Orchestration of the InstructLab flow like this should not be of concern to the individual libraries and instead be handled by the overarching InstructLab SDK which will maintain the user contracts. The InstructLab SDK will need to work within the bounds of what the Libraries expose as public APIs.

The benefit of the above is that the opinionated flow can be accessed in a more nuanced and piecemeal way while also gaining the potential for more advanced features. Say a consumer wants to:

1. Setup a custom config file for ilab (optional)
2. Initialize a taxonomy
3. Ensure their taxonomy is valid
4. Ingest some data for RAG and SDG (SDG coming soon)
5. Generate synthetic data using an InstructLab pipeline
6. Do some custom handling per their use case
7. Fine-tune a model using the custom config they initialized for their hardware
8. Evaluate their model after training using various benchmarks

A user could do this if they had an SDK.

(the structure of the SDK and actual arguments is discussed below)

However today, users are forced to run a sequence of commands tailored to only work with the proper directory structure on the system.

## Major Goals

1. Modularize the InstructLab workflow such that any part can be run independently
2. Allow users to choose whether or not to take advantage of the config/system-profile method of running InstructLab. Meaning they do not need any pre-existing configuration to run the SDK.
3. Standardize user contracts for the existing functionality of the InstructLab workflow. Existing CLI commands should be using the SDK once past click parsing, not separate code.
4. Define Contracts loose enough that functionality can be expanded as more advanced features are released.
5. Document SDK usage in upcoming InstructLab releases.

## Non-Goals

1. Exposing all library functionality immediately
2. Replacing CLI
3. Shipping an SDK that is generally available as opposed to v1alpha1 or v1beta1.

## Design

### Versioning

The SDK would start at version v1alpha1 such that it can change/break at any time for the first few iterations as libraries adjust their API surface.

### Structure

This SDK should live in a net new package inside of `instructlab/instructlab` preferably to limit unnecessary imports in a new repository. The SDK could be imported as `instructlab.core...`

The user surface initially should look like this:

`instructlab.core` contains all SDK definitions. Users can `from instructlab.core import...` to use specific SDK classes

For most of the existing InstructLab command groups, there should be a class:

`from instructlab.core import, Config, Taxonomy, Data, Model, RAG, System`

The full list of classes and their methods for now (subject to change during development process):

```console
instructlab.core.Config
instructlab.core.Config.init
instructlab.core.Config.show (get)
instructlab.core.Taxonomy
instructlab.core.Taxonomy.diff
instructlab.core.System
instructlab.core.System.info
instructlab.core.Data
instructlab.core.Data.ingest
instructlab.core.Data.generate_data
instructlab.core.Model
instructlab.core.Model.serve
instructlab.core.Model.train_model
instructlab.core.Model.process_data (calling the training library's data process class in a safe way)
instructlab.core.Model.evaluate_mt_bench
instructlab.core.Model.evaluate_dk_bench
instructlab.core.Model.evaluate_mmlu_bench
instructlab.core.RAG.ingest
instructlab.core.RAG.convert
```

a brief example:

```python

from instructlab.core import Config, Taxonomy, Data, Model

config_object = Config.init(...)
diff = Taxonomy.diff()

if diff:
    data_client = Data(data_path="", teacher_model="", num_cpus="", taxonomy_path="",)

    # not in v1alpha1
    data_path = data_client.ingest()
    # not in v1alpha1

    openai_compat_client = some_server()

    data_jsonls = data_client.generate_data(client=openai_compat_client, data=data_path)

    some_custom_handling(data_jsonls)

    # you can either use a config obj or pass trainer args
    model_client = Model(student_model=path_to_student_model, configuration=config_object)

    model_path = model_client.train_model()

    # since we initialized the model client with the config, the training args are passed implicitly
    eval_output = model_client.mt_bench(model_path=model_path)
```

The above example utilizes the configuration object to instantiate the `Model` class. However, a user could also pass `training_args=` directly to `model_client.train_model` to override the configuration class defaults. This allows the SDK to utilize the System Profiles of the ilab CLI but not rely on them too much.

Presumably, the distinct methods under each class will grow, which is why I am opting to make very distinct classes per command group. Another benefit to the parent classes is that individual methods can inherit defaults from the instantiation of the object.

These initial exposed functions can expand to include any new functionality that is more SDK oriented from the various libraries. For example, if SDG adds something like subset selection, teacher as annotator, data mixing, etc we could expose an `instructlab.core.Data.annotate` or `instructlab.core.Data.mix` that could be invoked in sequence in a user's script with other parts of the ilab workflow. Some things make _less_ sense to be exposed via a CLI, but still are critical to ensuring users get a good model and properly generated data.

There are certain things that only exist in `ilab` currently and functionality that is going to be moving these such as data ingestion, RAG, etc. Forming an SDK for `instructlab` allows us to capture all of these concerns under one API.

These endpoints in combination with the curated InstructLab Config File will open up these workflows to users and allow InstructLab to be easily incorporated into other projects. Allowing people to run things like data generation, and full fine-tuning via an SDK that pulls in their pre-existing `config.yaml` but also can be run independently will open new avenues for InstructLab adoption and extensibility.

## Changes to the CLI

The `ilab` CLI will need to adapt to this new structure. Commands like `ilab data generate` should, in terms of code, follow this flow:

1. `src/instructlab/cli/data/generate.py`
2. `src/instructlab/data/generate.py`
3. `src/instructlab/process.py`
4. `src/instructlab/core/data/generate.py`

So generally: cli -> process management package to kick off a sub-process -> internal handling package -> core SDK (public definitions) -> library code, is the flow.

The flow of the CLI today is such that the cli package for a command (`src/instructlab/cli/data/generate.py`) parses the command line options, manages creating a sub-process, and passes control to the core code (`/src/instructlab/core/data/generate.py`). This then calls out to the library APIs

The internal handling package is necessary as it allows us to split off a sub-process when it makes the most sense for us before calling the library code directly. This is how the CLI works today.

The difference with an SDK is that we would eventually want to end up executing `core/data/generator.py`, the actual publicly consumable python SDK. This will ensure that the CLI can do whatever custom handling it needs to do on top, but eventually it must boil down to the `core` package which uses publicly available methods from the various libraries.

## Scope of work

In upcoming releases the InstructLab team should aim to:

1. Design the SDK given the structure above
2. Converse with Library maintainers to negotiate user contracts
3. Begin work to re-architect how the CLI works using the SDK
4. Publish an alpha SDK for public consumption

After this initial work, the team can scope adding net new functionality that is not in the CLI to the SDK.