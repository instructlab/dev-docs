# Publishing Strategy for InstructLab

This document describes the publishing strategy used for all models in the InstructLab HuggingFace [organization](https://huggingface.co/instructlab) and the InstructLab Ollama [user account](https://ollama.com/instructlab).

## What are we publishing and why?

The InstructLab team will be periodically training the full unquantized model with new Pull Requests to the [taxonomy](https://github.com/instructlab/taxonomy) repository. When the evaluation shows that the model has improved, the team will be publishing 4-bit quantized GGUF form model to a platform called Ollama and both an unquantized model and a 4-bit quantized GGUF form model to a platform called HuggingFace.

## What is Ollama?

[Ollama](https://ollama.com/) is a platform that allows users to run open-source large language models locally on their machines. Ollama covers a broad spectrum of users, from seasoned AI professionals to people looking to explore the potential of AI and makes it easier for users to leverage the power of LLMs without having to rely on a cloud infrastructure. Since Ollama only supports publishing GGUF models, we will only be publishing the 4-bit quantized versions of our Merlinite and Granite models to the InstructLab Ollama user account.

## What is HuggingFace?

[HuggingFace](https://huggingface.co/) is a centralized web service platform, similar to GitHub, for hosting Git-based repositories related to data science and machine learning. In the context of InstructLab, HuggingFace is the platform where we will be publishing releases of our model for consumption by the community.

We will be publishing two different kinds of model families - Merlinite and Granite.

## Merlinite

The Merlinite model family is based off the [Mistral](https://mistral.ai/) model family and uses the [Large-scale Alignment for chatBots (LAB)](https://arxiv.org/abs/2403.01081) alignment. You can read more about it [here](https://huggingface.co/instructlab/merlinite-7b-lab).

The InstructLab organization will be publishing a community version of the Merlinite 7B size model, in both unquantized and 4-bit quantized GGUF form to HuggingFace and just the 4-bit quantized GGUF form to Ollama.

## Granite

The Granite model family is the [foundational model family](https://www.ibm.com/downloads/cas/X9W4O6BM) for the IBM watsonx AI platform, designed for usage in a business environment. You can read more about it [here](https://huggingface.co/instructlab/granite-7b-lab).

The InstructLab organization will be publishing a community version of the Granite 7B size model using the [Large-scale Alignment for chatBots (LAB)](https://arxiv.org/abs/2403.01081) alignment, in both unquantized and 4-bit quantized GGUF form to HuggingFace and just the 4-bit quantized GGUF form to Ollama.

## HuggingFace Naming Scheme

The naming scheme for both Merlinite and Granite will follow this generic scheme:

`<model family> - <size> - <type (optional)> - <alignment > - <optional elements with more detailed info>`

The specific schemes that will be published are detailed below:

| Model Family, Size, Alignment, etc. | Release Branch Format | Purpose |
| --- | --- | --- |
| [`merlinite-7b-lab`](https://huggingface.co/instructlab/merlinite-7b-lab) | `release-yyyymmdd` | Where the full precision Merlinite safetensors live |
| [`merlinite-7b-lab-GGUF`](https://huggingface.co/instructlab/merlinite-7b-lab-GGUF) | `release-yyyymmdd` | Where the full precision and quantized Merlinite GGUFs live |
| [`granite-7b-lab`](https://huggingface.co/instructlab/granite-7b-lab) | `release-yyyymmdd` | Where the full precision Granite safetensors live |
| [`granite-7b-lab-GGUF`](https://huggingface.co/instructlab/granite-7b-lab-GGUF) | `release-yyyymmdd` | Where the full precision and quantized Granite GGUFs live |

## Ollama Naming Scheme

The naming scheme for both Merlinite and Granite will follow this generic scheme:

`<model family> - <size> - <type (optional)> - <alignment>`

The specific schemes that will be published are detailed below:

| Model Family, Size, Alignment, etc. | Release Tag Format | Purpose |
| --- | --- | --- |
| [`merlinite-7b-lab`](https://ollama.com/instructlab/merlinite-7b-lab) | `release-yyyymmdd` | Where the full precision and quantized Merlinite GGUFs live |
| [`granite-7b-lab`](https://ollama.com/instructlab/granite-7b-lab) | `release-yyyymmdd` | Where the full precision and quantized Granite GGUFs live |

## Retention Policy

The InstructLab team will maintain the most recent **10** published versions of the respective models.
