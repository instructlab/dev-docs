# Introduce Functionality to Utilize the InstructLab backend: `ilab` Profiles, and Command Adaptations

This document describes adding different data generation, mixing, and training configurations to the `ilab` CLI to enable higher-fidelity training using the backend code. By higher-fidelity, we mean models that perform better, were trained on better hardware, off of larger data sets, and used more intensive training techniques.

Users will be able to set a pre-defined profile type in their config.yaml that would enable sane defaults for higher fidelity training, generation, and evaluation. These defaults will funnel into existing and new flags for the CLI.

## `ilab` Profiles

### Building off of the InstructLab Structure Redesign

Commands like: `ilab model train` will still be a single command offered to users. However, new "profiles" offered by the CLI will enable users to toggle what mode they are running `ilab model train` in.

Eventually, we will want to enable `ilab profile init` and `ilab profile set` commands that allow users to initialize a custom profile based on one of the pre-defined ones. This is not targeted for an upcoming release, though.

### Immediate Goals and Core Principles

For the near future, there will be upper level profiles that can be initialized via `ilab config init`.

Profiles should at first be static and not exposed to users. Internally, a profile would look something like:

```yaml
profile:
   train:
       gpus: 0-4
       epochs: 15
       effective_batch_size: 48
       save_samples: 2
       learning_rate: 1
       warmup_steps: 1000
       deepspeed:
           enabled: True
           deepspeed_offload: ["cpu"]
           quantize_dtype: ["nf4", "fp8"]
           deepspeed_config: foobar.json
       
```

These can be rendered as files or in code.

#### Where would this be stored?

Click has the ability to associate objects with the context passed around the command structure. The profile could be:

1. a ctx.obj additionally to the `ctx.obj.config` object. So this would be referenced as `ctx.obj.profile`.
2. the profile could live inside of the config obj, moving some of the existing generation options into a sub object called `profile`. so this would look something like `ctx.obj.config.profile` which would have sub-elements like `.training`, `.evaluation`, `.generation`. 

#### All new options will be flags with variable levels of expertise

Options that are introduced in new commands should exist in the profile AND as command line arguments via click. **Click has a way to hide arguments using an `active` option one can implement in a custom class**. So some of the new more "expert" level flags will either be hidden unless a specific profile is enabled.

#### Is this just for training?

No, adding user profiles for generation and evaluation would be ideal. The forthcoming backend libraries are taking an approach where it will be easiest to have pre-baked "config" objects that will feed into them. We need to be careful about how we introduce new concepts like this. Adding lower level intensive configurations to ilab is confusing to community users.

**PLEASE NOTE:**

**For immediate releases I would introduce the idea of a profile, a way to set these via `ilab config init` and hardcoded profiles that plug into key `ilab` commands namely training, generation, and evaluation.**

### Reasoning

The profile settings will be used as arguments for most if not all libraries being introduced to the `ilab` backend.

Plugging into hardware acceleration and multi-phase training is the logical next step for `ilab`. Ensuring we do this in a clean way that does not overload our current commands is also crucial. Many of the processes in the backend are confusing so we want to abstract some of the steps away from users while also giving them a reasonable amount of choice in configuring these new processes. However, maintaining the current laptop story is important to users without hardware access. Splitting these two paths into separate profiles clearly defined for users maintains the integrity of each.

#### Single Command Approach Across Fidelity Levels 

Having a single entrypoint in terms of each command is a safe bet, since making different training, eval, and generation commands for each technique we introduce will scale poorly. `ilab model train`, for example, after setting a profile will account for most of the use cases `ilab` targets. The same goes for `ilab data generate` and `ilab model evaluate`.

## Representation of the Profile in the config.yaml

Adding onto the current config.yaml, adding a profile would be as simple as:

```
chat:
  context: default
  greedy_mode: false
  logs_dir: data/chatlogs
  max_tokens: null
  model: models/merlinite-7b-lab-Q4_K_M.gguf
  session: null
  vi_mode: false
  visible_overflow: true
general:
  log_level: INFO
profile: multi-gpu
serve:
  gpu_layers: -1
  host_port: 127.0.0.1:8000
  max_ctx_size: 4096
  model_path: models/merlinite-7b-lab-Q4_K_M.gguf
``` 
Notice I am proposing that if a profile is specified, the generation defaults move inside of the profile. This will be discussed more below.

Also, there should most likely be a default **cpu** profile always set if the user does not choose a different one.

## Proposal for Default Profiles

1. **CPU Only, Laptop Profile**
    - low instruction number for generation
    - gpus: -1 for training, eval, generation
    - no deepspeed config or accelerator config for training
    - low epoch number for training
    - single phase training, no eval support given that there are no GPUs.
2. **Single GPU, Generate, Train, Eval profile -- CUDA**
    - mid level instruction number for generation, higher than the default 10 (100-500)
    - eventual deepspeed support with LoRA and QLoRA in training
    - 10+ epochs for training
    - eval on checkpoints if hardware permits (depends on vRam)
3. **Multi GPU, Generate, Train, Eval profile -- CUDA**
    - high instruction count, 500 or so for generation
    - deepspeed support without quantization or LoRA
    - evaluation on checkpoints after a phase of training

**Profile 3 would most likely have sub-profiles for different GPU support.**

These initial pre-baked profiles will allow for most of the dynamic range we are looking for in upcoming releases.

## Workflows using profiles

1. `ilab config init --profile cpu_only`
2. `ilab model generate`
3. `ilab model train`

to use multi-gpu:

1. `ilab config init --profile multi_gpu`
2. `ilab model generate`
3. `ilab model train --model-dir=./phase00/model --data-dir=./phase00/data`
4. `ilab model evaluate --benchmarks=mmlu --input-dir=./phase10/model --output-dir=./phase10/data`

the above multi-gpu is a good example of where the profiles come into play. If the user had not set their profile, they would need to specify flags like:

`ilab model generate --num-instructions=1000 --num-cpus=15`
`ilab model train --devide=cuda --gpus=0:4 --epochs=15 --config=training_config.json --accelerator=deepspeed --deepspeed_config=ds_config.json --model-dir=./phase00/model --data-dir=./phase00/data --model-repo=instructlab/granite-7b-lab`
`ilab model evaluate --benchmarks=mmlu --gpus=1-2 --data-dir=/path-to-gen/mixed --tasks=["mmlu.algebra"...] --input-dir=./phase10/model --output-dir=./phase10/data`
...

Adding the CPU only profile will be really beneficial to `ilab` users as it will eliminate confusion over some common use cases.

Most of the time choosing these flags is not clear. For expert users this is where `ilab profile init` would come in for future release. They would have a custom profile with bespoke options. However, for most other users a pre-baked single GPU profile will be very useful. They could also for now just override with flags.

## Adaptations to Existing Commands as well as New Commands for Profiles and the Backend Code

`ilab` is gaining new functionality to plug into the backend pipeline. Exposing this in a way that preserved the current UX but also enables power users is a core design principle.

### Implementation specifics for `ilab model train`

`ilab model train` will maintain all of its current flags which will represent overrides for the values stored in the training section of a user profile. This means all of the existing flags will be ported to a default config with sane values set for all that are applicable.

The existing code which runs the HF trainer will be ported to another library which can also run multi-phase training. This library will be shelled out to using torchrun.

#### The Importance of a User Profile: Future Mixing and Matching of Training Techniques

Trying to find a middle ground between server use cases and laptop use case is important. Exposing an eventual way to mix and match different training techniques makes sense for the following use cases:

1. Developer with a Gaming PC:
   * Transformers+Pytorch support QLoRA && FSDP. While deepspeed might be a more "server-rack" use-case, having multi-phase training in the CLI for anyone with a consumer GPU makes sense.
2. Someone interested in ML, has a Homelab, or *anything with 2 GPUs*
   * Transformers+Pytorch supports DeepSpeed on a single system spreading the training over the GPUs. Any professional or hobbyist that has 2 GPUs will be looking for this option
3. The laptop use case
   * Maintaining QLoRA as the performant training mode for the laptop is crucial as most environments cannot handle the full models. However, unlocking some better results by using FSDP+QLoRA could improve local results and get people more interested in InstructLab.

The transformers library supports the following combinations:

1. LoRA + BitsNBytes Quantized Model + DeepSpeed (QLoRA + DeepSpeed)
   - someone with a consumer GPU could get much better training results from using this combo as opposed to torch without deespeed and inter-checkpoint eval
2. LoRA + FSDP
3. LoRA + DeepSpeed
   - This enables native multi-GPU support out of the box. Model Loading, trainer setup, and deepspeed initialization are all handled by transformers. I have tested this and it works great on something like 2x4090 (3090 too) or 2xA10

**NOTE**: Changing the way we use the transformers library is not in scope for upcoming releases. This document serves as a proposal for upcoming and future enhancements to training, generation, and evaluation. the notes above regarding combining LoRA, DS, FSDP, etc are meant to guide us in future training enhancements and to describe how there are too many options to add as conflicting flags.

Given these requirements a full profile internally for training would look like:

```yaml
profile:
   train:
       gpus: 0-4
       epochs: 15
       effective_batch_size: 48
       save_samples: 2
       learning_rate: 1
       warmup_steps: 1000
       deepspeed:
           enabled: True
           deepspeed_offload: ["cpu"]
           quantize_dtype: ["nf4", "fp8"]
           deepspeed_config: foobar.json
       
```

all available options are:

```yaml
model_path: str
data_path: str
output_dir: str

num_gpus: int
max_seq_len: int
max_batch_len: int
num_epochs: int
effective_batch_size: int
save_samples: int
learning_rate: float
warmup_steps: int

deepspeed:
    enabled: bool
    deepspeed_config: str
    ds_offload_strat: Enum["cpu", "nvme", None]
    cpu_offload_optimizer: bool
    cpu_offload_params: bool
    quantize_dtype: Enum["nf4", "fp8", None]
    
lora: 
    enabled: bool
    lora_rank: int
    lora_alpha: float
    lora_dropout: float
    target_modules: list
```

These options will be in the default profiles AND will be available as flags in the CLI. This may seem like a lot of flags to be adding, but as mentioned above some of these will be hidden behind "expertise" levels.

NOTE:

**Many of these are contingent upon one another, the CLI will validate a user profile when it is set. The CLI will error out if a profile is invalid.**

### Implementation Specifics for `ilab data generate`

**The below flags are an assumption and I need confirmation from the SDG team**

`ilab data generate` configuration should also be placed into top level profiles. The end result will look like:

```yaml
profile:
   train:
       gpus: 0-4
       epochs: 15
       accelerator: deepspeed
   generate:
       gpus: 1-2
       num_instructions: 1000
       taxonomy_path: /path/to/large/taxonomy
       num_grounded_questions: 10
       num_samples: 10
       config: path_to_more_sdg.yaml
       ...
```

Some of the new flags and profile options are:

- --gpus: str, defined which GPUs to use for generation
- --num-grounded-questions: int
- --num-samples: int
- --config: str, points to a config which outlines internal SDG options outside of the CLI

#### This is currently _partially_ in config.yaml... why would we change that?!

A lot of this exists in the current `config.yaml` structure. As compared to the other entities in config.yaml, the generation settings stand out as CLI options placed into the config for convenience. In this case convenience is not the same as clarity of use. `ilab init` populates these for most users and then they leave them be.

Opting to move generation options into the profile is preferred and more verbose in the sense that if someone is setting a profile, they are aware of the settings being enabled. Of course, readers may be wary of moving them entirely out of the config.yaml. I will outline the alternative of embedding the profile **into** the config.yaml below.

#### Following the above design principles

A new concept is an additional `config` option within the generation profile which points to optional configuration yaml that contains more data generation settings, this can live as a file reference in the profile. At first, this will be hardcoded to a single default. Eventually if a user wants to override things like the default prompts, we should allow custom configs. 

Mixing of the generated data before training will occur as part of generation. If we choose to expose options for data mixing to the users, these would be some of the options we need:

- --num-util-proc=int (?)
- --output-dir=str (defaults to generated/mixed)
- --knowledge-recipes=[]str (path to yaml)
- --skill-recipes=[]str (path to yaml)

These can be introduced to either the profile where applicable or just as flags if they are highly dynamic. Reminder, everything introduced in these new commands will also be CLI flags.

**So, a generation profile would have the following entries**:

```yaml
    generate:
       model_family: mixtral
       gpus: 1-2
       chunk_word_count: 1000
       num_instructions: 1000
       model_endpoint_url: 127.0.0.1:8000
       taxonomy_path: /path/to/large/taxonomy
       num_grounded_questions: 10
       num_samples: 10
       config: path_to_more_sdg.yaml
```

**If mixing is to be exposed that process would require a profile like**:

```yaml 
    generate:
       model_family: mixtral
       gpus: 1-2
       chunk_word_count: 1000
       num_instructions: 1000
       model_endpoint_url: 127.0.0.1:8000
       taxonomy_path: /path/to/large/taxonomy
       num_grounded_questions: 10
       num_samples: 10
       config: path_to_more_sdg.yaml
       mix:
           num-util-proc: 10
           output-dir: generated/mixed
           knowledge-recipes: k_recipes.yaml
           sill-recipes: s_recipes.yaml
```

You will notice some of the currently existing flags have been added to the profile like chunk_word_count and endpoint_url. Any flag that is relatively constant or can be viewed to have a sane default should live in the profile. Remember, all of these can be overriden via flags by expert users.

### `ilab model evaluate`

InstructLab will run benchmarks for model performance in upcoming releases. These benchmarks are useful for inter-checkpoint evaluation and full scale evaluation at the end of multi-phase training.

As of writing the two benchmarks are MMLU Bench and MT Bench. You might only want to run certain benchmarks depending on the particular changes made to the model. Ex: MMLU is useful for knowledge additions.

MMLU bench needs the following options:
- --model: str, default is granite (?)
- --tasks: []str, default is {"mmlu_pr"}. This is the list of MMLU tasks to run.
- --few-shots (int)

MT bench needs the following options:
- --api-key (?)
- --model: str, default is granite (?)
- --endpoint: str, vLLM compatible endpoint to use for the benchmark

Some other flags would be:
- --output-dir: str, determines where the best checkpoint is put for the next phase of training
- --input-dir: str, takes the directory of the model/checkpoint to evaluate.
- --benchmarks: []str, this will let the user specify which benchmarks in their profile to run
- --prune: bool, this will let the user prune the other checkpoints not evaluated as "best" when running eval in between training phases
- --gpus: number of GPUs to use. Internally called batch-size for MMLU bench


All of these options will be in the profile but can also be overriden by flags if needed. 

Here is an example profile specifying settings for mmlu benchmarking:

```yaml
profile:
   train:
       gpus: 0-4
       epochs: 1
       ...
   generate:
       gpus: 1-2
       num_instructions: 1000
       taxonomy_path: /path/to/large/taxonomy
       num_grounded_questions: 10
       num_samples: 10
       ...
   evaluate:
       prune: True
       input-dir: /models/foobar
       outpur-dir: /bench/output
       gpus: 3-4
       mmlu:
           few-shots: 1
           model: granite  
           tasks: "mmlu-pr"
       ...
```

a user could then simply run something like `ilab model evaluate --input-dir=foobar --benchmarks=mmlu`

checkpoint evaluation and full evaluation are different but ultimately run similar benchmarks just on different types of data. So a single command here makes sense. The profile will simply outline the config for the supported benchmarks.

The command will output a dictionary of scores from evaluation as well as pairs of questions and answers depending on the benchmark run. The scores will be shown to the user, and the data will all live in the `output-dir`.

Training depends on a specific directory format to pick up a checkpoint to resume from after evaluation is run. The specifics of that will be coordinated by the training library writers and the evaluation library writers, then implemented by the CLI.

**A full profile for evaluation would have the following structure**:

```yaml
       gpus: 3-4
       mmlu:
           tasks: "mmlu-pr"
           few-shots: 1
           model: granite
           data-dir: generated/mixed
           
       mt:
           model: granite
           endpoint: 127.0.0.1:8080
           
```

**Question: how will the library know which form of eval to run?**
