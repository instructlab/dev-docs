# Modifying the ilab config structure to be more streamlined

Currently, the `config.yaml` is large with major expansions in training, evaluation, and serving. Expansions help the user understand what the various configuration options are for each of these commands. However, there is a fine line between a verbose config and a cluttered one.

This document describes a new structure for various parts of the config.yaml, and begins to outline how there are different levels of expertise in `ilab` which should dictate which options are available by default.

## `additional_arguments` as a field in training, serving and evaluation

Taking a look at the current training config it looks like: 

```yaml
train:
  torch_args:
    nnodes: 1
    node_rank: 0
    nproc_per_node: 1
    rdzv_endpoint: 127.0.0.1:12222
    rdzv_id: 123
  train_args:
    chat_tmpl_path: /home/ec2-user/instructlab/venv/lib64/python3.11/site-packages/instructlab/training/chat_templates/ibm_generic_tmpl.py
    ckpt_output_dir: checkpoints
    data_output_dir: train-output
    data_path: ./taxonomy_data
    deepspeed_options:
      cpu_offload_optimizer: true
      cpu_offload_optimizer_pin_memory: false
      cpu_offload_optimizer_ratio: 1
      save_samples: null
    effective_batch_size: 100
    is_padding_free: false
    learning_rate: 2e-6
    lora:
      alpha: 32
      dropout: 0.1
      quantize_data_type: nf4
      rank: 2
      target_modules:
      - q_proj
      - k_proj
      - v_proj
      - o_proj
    max_batch_len: 1000
    max_seq_len: 96
    mock_data: false
    mock_data_len: 0
    model_path: instructlab/granite-7b-lab
    num_epochs: 1
    random_seed: 42
    save_samples: 100
    warmup_steps: 10
```
While useful and clear to the user, this config is hard to maintain, and most users will not care about a large portion of the options.

Keeping some of the key options like `num_epochs`, `deepspeed`, `lora`, and key directories a possible training config could look like:

```yaml
train:
  train_args:
    ckpt_output_dir: checkpoints
    data_output_dir: train-output
    data_path: ./taxonomy_data
    deepspeed_options:
      cpu_offload_optimizer: true
      cpu_offload_optimizer_pin_memory: false
      cpu_offload_optimizer_ratio: 1
      save_samples: null
    learning_rate: 2e-6
    lora:
      alpha: 32
      dropout: 0.1
      quantize_data_type: nf4
      rank: 2
      target_modules:
      - q_proj
      - k_proj
      - v_proj
      - o_proj
    max_batch_len: 1000
    model_path: instructlab/granite-7b-lab
    num_epochs: 1
    save_samples: 100
    warmup_steps: 10
    additional_arguments: ["--is-padding-free=False"...]
```

`additional_arguments` holds the rest of the training arguments. `ilab` would validate these against an internally maintained list of supported options before passing to the respective library.

The same structure can be applied easily to the serve config.Currently this config looks like: 

```yaml
serve:
  backend: ''
  host_port: 127.0.0.1:8000
  llama_cpp:
    gpu_layers: -1
    llm_family: ''
    max_ctx_size: 4096
  model_path: models/merlinite-7b-lab-Q4_K_M.gguf
  vllm:
    vllm_args: []
```

This has the opposite problem as training. The key here is that neither of these options are necessarily wrong, but to have both the verbose structure in the training config juxtaposed against the practically hidden structure of `vllm_args` is not ideal design practice. If we could merge the two approaches to use a common design language that exposes enough key arguments which are commonly edited while also not making the config confusing, this is what we should aim for.

Being very general, this would look something like:

```yaml
serve:
  backend: 'vllm'
  host_port: 127.0.0.1:8000
  max_ctx_size: 5120
  gpus: 2
  llm_family: ''
  model_path: models/merlinite-7b-lab-Q4_K_M.gguf
  served_model_name: "merlinite"
  additional_arguments: ["--block-size=16", "--dtype=fp8"...]
```

Backends like vllm have a large amount of command line options. Adding each and every one of these to our config.yaml is out of the question. However, supporting a large amount implicitly via additional_arguments is a good compromise. Additionally, the above structure lets us choose which ones we think deserve a spot in the config and which are more uncommon or preserved for power users.

This structure also allows us to flatten the config, something that is beneficial for flag mapping and config parsing. Options like max_ctx_size can apply to both vllm and llama-cpp. Options that are only applicable to a singular backend can be validated internally. Nested configurations within our config.yaml create a barrier both for the users and for flexible parsing of the config within `ilab`. additional_arguments will hopefully allow us to move generally away from nested configurations in both training and serving. 
