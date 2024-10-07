# Design Proposal - HF Accelerate

## Overview

InstructLab training is currently being required to support two distributed sharding frameworks: DeepSpeed and FSDP1. Additionally, there is a future plan to adopt FSDP2 once the project matures. Each of these frameworks has its own process for preparing models, optimizers, sharding, and all of their own custom options.

With all this in mind, in order to maintain a simple common codebase, an abstraction for sharding frameworks is required. This is why we have pulled in Hugging Face Accelerate as a lightweight sharding abstraction, to enable a common interface for sharding frameworks with pluggable configs, and to avoid overly-branching code paths.

## How it is being used

(maybe insert diagram here)

### How it is implemented in code

Accelerate usage consists of a single import:

```python
from accelerate import Accelerator
```

We begin by setting up our accelerator object via our `setup_accelerator` function:

```python
accelerator = setup_accelerator(args, model, grad_accum)
```

This checks the selected sharding framework, and sets up the appropriate config:

```python
def setup_accelerator(args, model, grad_accum):
    if args.distributed_training_framework == "deepspeed":
        # Third Party
        from deepspeed import DeepSpeedEngine
        ...
        accel_args = {
            "deepspeed_plugin": get_ds_plugin(...),
        }
    elif args.distributed_training_framework == "fsdp":
        accel_args = {
            "fsdp_plugin": get_fsdp_config(args, model),
        }
    else:
        raise ValueError(
            f"Unknown sharding framework: {args.distributed_training_framework}"
        )
    accelerator = Accelerator(
        **accel_args,
    )
    accelerator.even_batches = False
    return accelerator
```

Now this Accelerator object can act as a universal sharding framework config, and we can prepare our training objects accordingly:

```python
model, optimizer, _, lr_scheduler = accelerator.prepare(
    model,
    optimizer,
    deepcopy(train_loader),
    lr_scheduler,
)
```

As an additional bonus, the accelerator object allows us to do universal checkpoint saving and resuming as well. For saving model checkpoints, we can simply use:

```python
accelerator.save_model(
    model,
    save_directory=output_dir,
    max_shard_size="5GB",
    safe_serialization=True,
)
```

and to save full state for resuming training, we can use:

```python
accelerator.save_state(
    output_dir=output_dir,
    safe_serialization=True,
)
```

## The immediate benefit

The inclusion of Accelerate in this manner drastically simplifies the process of supporting multiple sharding frameworks. Rather than having diverging code paths for model setup, optimizer setup, sharding configuration, distributed initialization, state saving, and model checkpointing, all of these training steps can be supported with the same code. The common abstraction allows us to maintain both DeepSpeed and FSDP, and prepares us for inclusion of additional sharding frameworks.

## Impact on overhead

### Performance

There has been no noticeable performance impact observed during development, but we will defer to the Performance and Scale team for final measurements.

### Usability

This makes the library easier to read, but also requires knowledge of an additional package. While there exists [documentation](https://huggingface.co/docs/accelerate/v1.0.0rc1/en/index), it does require some additional code exploration to actually understand what some functions (like `prepare`, `save_model`, etc.) are doing behind the scenes, making some processes less transparent than if implemented directly with `torch`.

While sharding framework code will not need to be reviewed as thoroughly, this does not change the fact that one still needs to understand the configuration options per framework and how they behave.

### Package Management

This inclusion requires one additional python package to be managed as a dependency. Currently, that package is `accelerate==0.34.2` but we plan to immediately upgrade to `accelerate==1.0.0` once it is moved from pre-release to official release.

## Long-Term Bonuses and Risks

The two clearest long-term bonuses are:

- Simplifies the process of onboarding additional sharding frameworks in the future, as well as deprecating existing sharding frameworks
- Vastly improves the readability and maintenance of our code by avoiding diverging paths for various sharding frameworks

There are, however, important risks to be considered. Ultimately, if we are confident that we wish to eventually stick with a single sharding framework in the future and contribute directly to that project, the risks of including Accelerate may outweigh the benefits. Please read the next section carefully to understand why, if Accelerate no longer provides a significant benefit to us, should be removed.

### Why Stop There?

With the inclusion of a Hugging Face abstraction library, this begs the question "why not keep going and pull in more of the HF stack? Won't it simplify things further then?"

It is important to reiterate here the clear downsides of HF Accelerate, and what we wish to avoid moving forward. Accelerate makes our code more opaque, and hinders our ability to easily understand and customize as needed without being reliant upon a third-party dependency. We become reliant upon HF bug fixes, release cycles, and documentation, and without an explicit member of the HF community to manage these for us, this introduces risk that has to be heavily considered.

For example, if we wanted to change how model saving worked, or a user wanted to simply understand how model saving worked, they would no longer be able to do so through our library directly, but instead would need to read HF documentation, and contribute back to HF repos with the hope that the customization is considered generally useful and approved. More niche or experimental customizations may not be approved at all, and would have to be patched in directly or overwritten in a rather hacky manner within our library. This as described is not the most dire of circumstances, as one could argue that if it is not ready to be included upstream, it may not be ready to be included in our library. Where things get more serious, however, is with the introduction of bugs. At this very moment, there is a bug in `v0.34.2` that means FSDP model saving does not behave as expected. This fix is resolved in `v1.0.0rc1`, but there is no official release yet that can be pulled in with this fix. So for instructlab-training `v0.5.x`, we are currently maintaining a patch directly in our library to fix the bug while working with `v0.34.2`, and it will sit there temporarily until Accelerate has a new official release.

This expands well into the wider topic of dependency management. A package like Accelerate will have frequent updates and changes moving forward, and we will need to ensure consistent compatibility between our training library releases and the required Accelerate versions. A new dependency also means shipping with another package, with a large set of code. While in most cases this is harmless, a package that is used directly on top of our work in place of our existing code, adds a significant amount of bloat for developers and users alike. The requirement shifts from understanding a line of code, to understanding a full code path behind a line of code, the quality of which cannot be guaranteed by our engineers.

Ultimately, these risks and overhead management costs make sense for Accelerate, because the package provides significant benefit that far outweigh our concerns. When considering additional HF packages, however, there is currently **no obvious benefit** to their inclusion, which only further annunciates the risks associated. It must be made clear that the risks of Accelerate are non-negligible, and that the conveniences and positive impact provided are what push it over the line of inclusion. It is by no means a "no brainer" to further include any additional HF packages at this time, unless they afford us a similar benefit.
