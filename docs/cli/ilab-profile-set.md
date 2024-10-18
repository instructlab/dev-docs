# `ilab profile set`

### Usecase

1. User needs to set:
    a. Model Path for SDG, Eval, Serving, Training
    b. GPUs for SDG, Eval, Serving, Training
    c. Training Config Per-GPU (based on vram)
2. User wants us to detect all of the above for them and provide a sane config

### Workflow

The user will run `ilab profile set`
Alongside the various model paths and GPU amounts, this command will
set the train profile for the following scenarions:

    1. CPU
    2. single consumer GPU
    3. multi consumer GPU
    4. single server GPU
    5. multi server GPU
    6. MacOS (once MPS support is added)
    
There is also a Choose for me option which reads the Nvidia cards on the system and assigns a cfg+train profile based off the amount of vRAM

A general output when using the non-default `-i` interactive mode looks like this:

```
ilab profile set -i
How many Dedicated GPUs do you have?: 8
Please choose a system profile to use
[0] Choose for me (Hardware defaults)
[1] CPU
[2] Single Consumer GPU
[3] Single Server GPU
[4] Multi Consumer GPU
[5] Multi Server GPU
[6] MacOS
Enter the number of your choice [hit enter for the hardware defaults profile] [0]: 4
Please choose the GPU Training profile that best matches your system
[0] NVIDIA 2x A100/H100
[1] NVIDIA 8x L4
[2] NVIDIA 4x L40
[3] NVIDIA 4x A100/H100
[4] NVIDIA 8x L40
[5] NVIDIA 8x A100 or H100
Enter the number of your choice [hit enter for the hardware defaults train profile] [0]: 3
```
or if you have it choose for you:

```
ilab profile set
Detecting hardware...
We detected the profile to best match your system is the `Multi Server GPU` config with the `NVIDIA 4x A100` Training Profile. Please run the command again with `-i` if this is incorrect.
```
