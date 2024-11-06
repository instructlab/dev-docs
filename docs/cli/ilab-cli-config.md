# Design for `ilab` CLI Configuration

## Intro, Goal & Benefits

This document represents the next step in the evolution of the configuration of the `ilab` CLI.
Ever since `ilab` was created, it relied upon the existence of configuration file written in YAML.
The current state of the config is documented [here](https://github.com/instructlab/dev-docs/pull/132).

The goal of this document is to provide a vision and development roadmap for configuring `ilab`.

The benefits of the proposed design are:

* `ilab` no long needs a configuration file. `ilab config init` is no longer needed to run other `ilab` commands.
* A heirarchy of configuration overrides for `ilab`.
* A configuration file is now optional, user provided, and only contains the variables the user cares about overriding.
* The option for YAML, TOML, and JSON configuration files.
* Eliminating backwards compatability concerns for the configuration file between future versions of `ilab`.
* A configuration file layout that reflects individual commands.

## Development Stages

The proposed design can be broken down into two phases of development.

The first phase is focused on decoupling `ilab` from a configuration file and establishing a hierarchy of internal and user provided inputs for configuration.

The second phase builds on the first phase of of development and proposes a new configuration file layout, supplementary `ilab profile/config` commands and a new configuration file format.

### Proof of Concept Branch

The proof of concept of the work outlined in Phase 1 and Phase 2 can be seen and run on [this branch](https://github.com/alimaredia/instructlab/tree/config-enhancement).
This branch serves to show how much work was required for some of the changes described, and for a user to be able to run and get a feel for the options described below.

This branch has been created in such that each commit represents a sub phase of development described below. Users can run `ilab model chat`, `ilab model serve`, and `ilab system info` to see the changes reflected.
Each commit message also describes future work that would need to go into the sub phase.

### "Configuration" vs "Profile"

Throughout this document the terms "configuration", and "profile" will be used. The term "configuration" describes the entire internal data structure within `ilab` for configuration used when a command is run.

A "profile" is a subset of configuration needed to allow `ilab` to function optimally on a given piece of hardware. When applied a profile overrides the provided subset of configuration.

## Phase 1: Hierarchy of configuration sources

### Phase 1 Overview

Currently `ilab` needs a configuration file to function. The goal of this phase of development is to decouple `ilab` from needing a configuration file, and to establish a clear hierarchy for users to understand how their configuration is set.

Configuration overrides happen both internally within `ilab`, and with user provided overrides.

The order of these configuration overrides is:

1. (System Internal) DefaultConfig data structure AKA the system defaults
2. (System Internal) Hardware & System Profile auto-detection
3. (User provided) A configuration file
4. (User provided) Flags for a specific command to override specific configuration variables

### Phase 1.1: System Defaults

Phase 1.1 involves having a Pydantic BaseModel for the internal configuration and setting defaults for each value.
In the branch, the [DefaultConfig object](https://github.com/alimaredia/instructlab/blob/config-enhancement/src/instructlab/profiles/default_configuration.py) object serves this purpose. This data structure is based off of the [Config](https://github.com/instructlab/instructlab/blob/main/src/instructlab/configuration.py#L528) that currently exists in `ilab`.

### Phase 1.2: System Profile Auto-detection

Maximizing the effectiveness of Instructlab involves changing the configuration to specify specific models, and configuration parameters. These changes depend on the hardware `ilab` is running on.
[System profiles](https://github.com/instructlab/instructlab/pull/2495) have been created to provide users the best possible configuration for their hardware.

Though `ilab` has many configuration options, only a subset of them constitute a system profile. Take the following examples of the configuration overrides needed for the `chat` and `serve` sections for an Nvidia 4xL4 profile.
This profile is detected for Nvidia devices with 4 L4 GPUs.

```yaml
chat:
  model: instructlab/granite-7b-lab
serve:
  backend: vllm
  model_path: instructlab/granite-7b-lab
  vllm:
    gpus: 4
```

When the `nvidia-l4x4` profile is detected, only those configuration options are changed within the internal configuration data structure and the following message is given to the user:

```shell
$ ilab model chat
...
Set nvidia-l4x4 configuration
...
```

General hardware profiles are necessary to allow users to maximize hardware that hasn't had a custom profile created yet. These profiles are defined by total GPU VRAM and the chip maker.
For example the `nvidia-4xgpu-16gb` profile is detected for Nvidia devices with 4 GPUs with at least 16GB VRAM each.

```shell
$ ila model chat
...
Set nvidia-4xgpu-16 configuration
...
```

A general hardware profile can be very similar or identical to a custom profile. In the development branch, the `nvidia-4xgpu-16` and `nvidia-l4x4` profiles are identical.

Auto-detection can be skipped by passing the `--skip-profile-autodetection` flag to `ilab`.
```shell
$ ilab --skip-profile-autodetection model chat
Set default configuration
```

### Phase 1.3: User provided configuration file

Configuration in Phase 1.1 & 1.2 happens internal to `ilab`. Configuration in this phase and beyond is provided by the user.

Previously `ilab` configuration file users interacted with contained the entire instructlab configuration.

In this design, the configuration file looks the same way as a profile does. It contains just the values a user wants to override.

This file can either be placed into `$XDG_CONFIG_HOME/instructlab` as a file named `config.yaml` or by passing the file to the `-c` flag of `ilab`.

The following two examples apply the same configuration that changes the `model_path` for `ilab model serve`.

```shell
$ cat ~/.config/instructlab/config.yaml
serve:
  model_path: instructlab/merlinite-7b-lab
$ ilab model serve
```

```shell
$ cat config.yaml
serve:
  model_path: instructlab/merlinite-7b-lab
$ ilab -c config.yaml model serve
```

If a config file is passed in via the `-c` flag, that config is read. If no flag is passed in, then `ilab` looks in `$XDG_CONFIG_HOME/instructlab` for the configuration file.
It is not possible for more than one user provided configuration file to be read by `ilab`.

The configuration overrides are applied on top of the auto-detected profile (unless --skip-profile-autodetection is passed), not just the default configuration.

The user provided configuration file does not just have to be in YAML, it can also be in JSON or TOML. These files will be name `config.toml` and `config.json` accordingly.

```shell
$ cat config.toml
[serve]
model_path = "instructlab/merlinite-7b-lab"
$ cat config.json
{
  "serve": { "model_path":
    "instructlab/merlinite-7b-lab"
  }
}
```

If there is more than one configuration file in `$XDG_CONFIG_HOME/instructlab`, for example a `config.toml` and a `config.yaml` then neither config file is read and a warning is thrown to the user.

```shell
$ ls ~/.config/instructlab/
config.toml config.yaml
$ ilab model serve
...
WARNING found configuration files at /home/ec2-user/.confg/instructlab/config.toml and /home/ec2-user/.config/instructlab/config.yaml. Configuration files not applied.
...
```

### Phase 1.4: User provided flags

The last set of overrides are flags for individual commands. The values set in the flags for a specific command supercede all other configuration.
Flags pick up the values from the last applied configuration override and have them as their defaults.

```shell
$ cat ~/.config/instructlab/config.yaml
serve:
  model_path: instructlab/merlinite-7b-lab
$ ilab model serve --help
Set nvidia-l4x4 configuration. Run `ilab -v` for more information
Applied configuration from file at /home/ec2-user/.config/instructlab/config.yaml
Usage: ilab model serve [OPTIONS]

  Starts a local server

Options:
  --model-path PATH           Directory where model to be served is stored.
                              [default: /home/ec2-user/.cache/instructlab/mode
                              ls/instructlab/merlinite-7b-lab; config:
                              'serve.model_path']
...
```

Only a subset of all configuration variables are able to be set as flags. These flags are pre-determined by the maintainers of the commands themselves.

## Phase 2: Enhancements to the CLI configuration UX

### Phase 2 Overview

After the foundation has been established in the first phase of development, the second phase of development builds up it by introducing even broader changes to `ilab` configuration expierence.

NOTE: TOML and JSON file support for 1.3 and phase 2.2 have not been implemented in the prototype branch

### Phase 2.1 New config file layout

The current configuration file and format evolved from the initial `ilab` commands.
Those were commands such as `ilab generate`, `ilab train`, `ilab serve`, and `ilab chat`.

Today `ilab` commands have sub commands for each of the nouns `ilab` operates on (models, data, taxonomy, system, etc.).

A new configuration file format would mirror `ilab` commands, making it intuitive for users of to see which command their configuration variables belongs.

### Phase 2.1.1 Subcommand based file lay out

The following is an example of a config file layout based on the subcommands.

Current file:

```yaml
general:
  debug_level: 1
chat:
  model: instructlab/granite-7b-lab
serve:
  backend: vllm
  model_path: instructlab/granite-7b-lab
  vllm:
    gpus: 4
    max_startup_attempts: 120
```

Proposed change:

```yaml
global:
  debug_level: 1
model:
  chat:
    model: instructlab/granite-7b-lab
  serve:
    backend: vllm
    model_path: instructlab/granite-7b-lab
    vllm:
      gpus: 4
      max_startup_attempts: 120
system:
  info:
    new_var: foo
  profile:
    device: 'hpu'
```

In the current layout it is unclear where the hypothetical `new_var` for `ilab system info` command would go since a `system` section of the configuration file does not exist.

This proposed configuration also shows how easy it is to create configuration for a fictional `ilab system profile` command.

The `global` section of the config would be the same and would be for configuration that applies for the entire system like the log level.
This section exists with future features like workspaces in mind as a place for it's configuration.

### Phase 2.1.2 Addressing config file backwards compatibility

Since `ilab` does not depend on configuration file anymore to function, the config should not need to be backwards compatible. 

`ilab` should be able to take an invalid config and still run because of the profile auto-detection.

If commands or variables are removed between versions of `ilab` then `ilab` will throw warnings of invalid configurations variables or sections set in the config and apply those that are still valid.

Here is an example of a config with several invalid sections, an invalid configuration variable, and the output the proposed `ilab` design gives.

```shell
$ cat ~/.config/instructlab/config.yaml
model:
  serve:
    vllm_args:
      - --tensor-parallel-size=4
    biz: baz
  bar: boz
foo: buz
```

```shell
$ ilab -v
DEBUG 2024-11-19 17:58:09,455 root:106: Applying configuration provided or detected at /home/ec2-user/.config/instructlab/config.yaml
DEBUG 2024-11-19 17:58:09,456 instructlab.profiles.utils:79: model.serve.biz is not a valid configuration variable
DEBUG 2024-11-19 17:58:09,456 instructlab.profiles.utils:79: model.bar is not a valid configuration section
DEBUG 2024-11-19 17:58:09,456 instructlab.profiles.utils:79: foo is not a valid configuration section
Applied configuration from file at /home/ec2-user/.config/instructlab/config.yaml
```

Backward compatability would not be a concern between versions of `ilab` after this design is implemented.

Backward compatability concerns do exist for current `ilab` users with existing `$XDG_CONFIG_HOME/instructlab/config.yaml` in the format that reflects the older command structure.
There are many options for how to handle older formatted YAML configuration files. Some include:

* Renaming the auto-detected configuration file from `config.yaml/toml/json` to `ilab-config.yaml/toml/json`.
* Removing auto-detection of user provided configuration files all together. All user provided configuration files would need to be run with the `-c` flag to `ilab`.
* Automatically migrating a YAML configuration file in the older format to the newer format.

Further discussion and concensus will need to happen to decide what action will be taken.

### Phase 2.2 Supplementary `ilab profile/config` commands

The following commands would enhance the configuration experience.

### Phase 2.2.1 `ilab config show`

This command would dump the full configuration, configuration for a command group, configuration for a specific command, or the configuration for a specific variable.

```shell
$ ilab config show ilab.model.chat
model:
  # Configuration for `ilab model chat` is:
  chat:
    context: default
    greedy_mode: false
    logs_dir: /home/ec2-user/.local/share/instructlab/chatlogs
    max_tokens:
    model: /home/ec2-user/.cache/instructlab/models/merlinite-7b-lab-Q4_K_M.gguf
    session:
    vi_mode: false
    visible_overflow: true
```

```shell
$ ilab config show ilab.model.serve.model_path
model_path: /home/ec2-user/.cache/instructlab/models/merlinite-7b-lab-Q4_K_M.gguf
```

The value(s) shown would reflect the configuration after profile auto-detection and user provided configuration files are applied.

`ilab config show` could also be a starting point for new configuration files.

```shell
$ ilab config show ilab.model.chat
[ilab.config.chat]
context = "default"
greedy_mode = false
logs_dir = /home/ec2-user/.local/share/instructlab/chatlogs
max_tokens = 4096
model = /home/ec2-user/.cache/instructlab/models/merlinite-7b-lab-Q4_K_M.gguf
session = "new"
vi_mode = false
visible_overflow = true
$ ilab config show ilab.model.chat > ~/.config/instructlab/config.toml
```

### Phase 2.2.2 `ilab config verify`

This command could be used to check if a user provided config file has any errors.

Here are some examples:

```shell
$ ilab config verify
Success! Config file at /home/ec2-user/.config/instructlab/config.yaml is valid
```

```shell
$ cat ~/.config/instructlab/config.yaml
model:
  serve:
    vllm_args:
      - --tensor-parallel-size=4
    biz: baz
  bar: boz
foo: buz
$ ilab config verify ~/config.yaml
Configuration file at /home/ec2-user/.config/instructlab/config.yaml contains 3 errors:
model.serve.biz is not a valid configuration variable
model.bar is not a valid configuration section
foo is not a valid configuration section
```

### Phase 2.2.3 `ilab config generate`

This command would generate a brand new blank config file with comments to get the user started.

```shell
$ ilab config generate
Empty config file generated at /home/ec2-user/.config/instructlab/config.yaml
$ cat ~/.config/instructlab/config.yaml
# This is an autogenerated config file
# This is section is for `ilab model` commands
model:
  # This is section is for `ilab model serve`
  serve:
    #model_path: instructlab/merlinite-7b-lab
# This is section is for `ilab data` commands
data:
  # This is section is for `ilab data generate`
  generate:
    #teacher_model: instructlab/merlinite-7b-lab
```

This version of the command would generate the empty configuration file in the `--output-dir`.

```shell
$ ilab config generate --output-dir ~/
Empty config file generated at /home/ec2-user/config.yaml
```

Users unfamiliar with TOML could be aided by comments in the configuration created by `ilab config generate`.

```shell
$ ilab config generate
Empty config file generated at /home/ec2-user/.config/instructlab/config.yaml
$ cat ~/.config/instructlab/config.yaml
# This is an autogenerated TOML config file
# To learn more about TOML visit:
# https://toml.io/en/v1.0.0#keyvalue-pair
# https://toml.io/en/v1.0.0#table

# For all config variables for a command run:
# `ilab config show ilab.command.name`
# Tables configure a specific `ilab` command
# ex: `ilab model chat`
# [ilab.model.chat]
# model = merlinite-7b-lab-Q4_K_M.gguf

# This section is for another `ilab` command
# [ilab.model.serve]
# backend = "vllm"
```

### Phase 2.2.4 `ilab profile get`

Gets the profile name that is auto-detected on the system. This requires the profile name to be in the `global` section of the files for auto-detected profiles.

```shell
$ ilab profile get
nvidia-l4x4
```

```shell
$ cat nvidia-l4x4.yaml
global:
  profile:
    name: nvidia-l4x4
model:
  chat:
    model: instructlab/granite-7b-lab
  serve:
    backend: vllm
    model_path: instructlab/granite-7b-lab
    vllm:
      gpus: 4
```
