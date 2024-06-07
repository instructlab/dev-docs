# Design for `ilab model serve` command with backend support

## Background

With the [request from the community](https://github.com/instructlab/instructlab/issues/1106) for `ilab` to serve different backends such as [vllm](https://docs.vllm.ai/en/stable/) and the [cli redesign](ilab-model-backend.md), this design doc's purpose is to flesh out the behavior of the `ilab model serve` command.

Specifically, this doc addresses the design of subcommands of `ilab model serve` that apply to
different serving backends.

## Design

### Backend

Since the subject of the `ilab model serve` command is a model, regardless of the format of the model, every command takes in the `--model` flag or uses its default value in the config.

`ilab model serve` has a new flag `--backend` that will be used to serve models with. As of this design, the two backends `ilab` would serve with are `llama-cpp` and `vllm`.

This would lead to the commands:

- `ilab model serve --backend llama-cpp`
- `ilab model serve --backend vllm`

There are specific flags for `ilab model serve` that would apply to all backends. These can be viewed by running `ilab model serve --help`.

The following is an overview for the flags of `ilab model serve`:

```console
ilab model serve
|
|_______ (backend agnostic flags)
|
|_______ --backend ['llama-cpp', 'vllm']
|_______ --backend-args
```

The `backend` flag will also be available as an option in the config file (`config.yaml`). This will allow users to
set a default backend for `ilab model serve` in the config. Also, commands like `ilab model chat`
and `ilab data generate` that serve models in the background will use the default backend specified
in the config. Here is an example of what the config file would look like:

```yaml
serve:
  gpu_layers: -1
  host_port: 127.0.0.1:8000
  max_ctx_size: 4096
  model_path: models/merlinite-7b-lab-Q4_K_M.gguf
  backend: llama-cpp
```

### Backend flags

The `--backend-args` flag is a string that will be passed to the backend as arguments. This flag is used to pass
backend-specific arguments to the backend. Multiple values will be supported, however the exact formatting will be
defined in the implementation proposal. The backend will be responsible for parsing individual arguments.

It will also be available as an option in the config file (`config.yaml`). This will allow users to set default backend arguments for `ilab model serve` in the config. Here is an example of what the config file would look like:

```yaml
serve:
  backend: llama-cpp
  backend_args:
    num_gpu_layers: 4
    max_ctx_size: 1024
```

For clarity and ease of implementation, when using the `--backend-args` flag, the user must pass the
`--backend` flag as well. This is to ensure that the backend-specific arguments are passed to the
correct backend. Any backend-specific arguments that are not passed to the correct backend will be
reported as an error.

## Command Examples

### Bare-bones but model specific command

```shell
ilab model serve --model <PATH>
```

- Serves the model at `<PATH>`.
- If the `<PATH>` is the path for a model that can be run by `llama-cpp` then `llama-cpp` is
  automatically used as the model serving backend. The current auto-detection logic will rely on a
  valid GGUF file format. If the model is a valid GGUF file, then `llama-cpp` will be used as the model serving backend.
- If the `<PATH>` is the path for a model that can be run by `vllm` then `vllm` is automatically used as the model serving backend.
- If the model at `<PATH>` can be run by either backend, then a default backend defined in the
  config will be used as the model serving backend. In the case where there is ambiguity and a setting is not defined, a hardcoded preference will be used (all currently supported providers do not have this issue). A future profile specification will likely replace the hardcoded fallback.

### Bare-bones command

```shell
ilab model serve
```

- This command has the same behavior as the one above but the `--model` is whatever the default model path is in the config. This is the existing behavior of `ilab serve` today.

### Llama-cpp backend specific commands

```shell
ilab model serve --model <PATH> --backend llama-cpp --backend-args '--num-gpu-layers 4'
```

- This command serves a model with `llama-cpp`.
- If the model provided is not able to be served by llama-cpp, this command would error out and suggest an alternate backend to use.
- The existing flags to `ilab serve` (besides `--model-path` & `--log-file`) are now specific to the llama-cpp backend.

### vllm backend specific commands

```shell
ilab model serve --model <PATH> --backend vllm --backend-args '--chat-template <PATH>'
```

- This command serves a model with `vllm`.
- If the path provided is not able to be served by `vllm`, this command would error out and suggest an alternate backend to use.
- There are [dozens](https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html#command-line-arguments-for-the-server) of flags for vllm. Whichever arguments the community deems the most important to include, will be added as flags to `ilab model serve vllm`.
- Any remaining arguments can be specified in the value of the flag `--vllm-args`.

## Testing

An additional end-to-end test will be added that for a new backend for `ilab model serve`. This new test should be triggered whenever code changes to the new backend serving code are made or before a release.

This new test will do the following:

1. Initialize ilab in a virtual env via `ilab config init`.
2. Download a model via `ilab model download`.
3. Serve the downloaded model with the new backend via `ilab model serve`.
4. Generate synthetic data using the served model via `ilab data generate`.
5. Chat with the served model via `ilab model chat`.
6. Any future commands that interact with a served model should be added to the test.

Some commands, like `ilab model chat` and `ilab data generate`, serve models in the background as part of the command. If automatic serving of a new backend is implemented for a command, testing of that command that will also be included in the new end-to-end test.

## Handling existing backend-specific commands

The existing `ilab model serve` command has flags that are specific to the `llama-cpp` backend. The current list of flags are:

- `--num-gpu-layers`
- `--max-ctx-size`
- `--num-threads`

These flags will be moved to `--backend-args` and will be used as the default arguments for
`llama-cpp` backend. This will allow for a more consistent experience across backends. The flag will
be supported up to two releases after the release of the new backend. After that, the flag will be
removed. During the two releases, a warning will be printed to the user when the flag is used.
