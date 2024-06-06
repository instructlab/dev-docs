# API Definitions Guidelines

This document describes how service APIs will be managed for `InstructLab` and the sub-components of the `InstructLab` backend.

## What parts of InstructLab need service APIs?

There are two primary classes of service APIs needed to support InstructLab:

* `Platform APIs`: These are APIs that are ignorant of `InstructLab` and provide generic AI platform capabilities (e.g., Fine Tuning, SDG, Eval)
* `InstructLab APIs`: These are the APIs that reflect the user-facing functionality of `InstructLab` itself. They are aware of the end-to-end `InstructLab` workflow.

The `InstructLab APIs` are essential for hosting `InstructLab` as a service in a repeatable way. The `Platform APIs` are critical for component reuse and extensibility (e.g., new SDG algorithms for new taxonomy data types), but they are not strictly required for hosting `InstructLab` as a service.

## How will service APIs be defined?

Service APIs will be defined using [OpenAPI](https://www.openapis.org/) format in [YAML](https://yaml.org/). For structural and style guidelines, see [api-definitions](../../api-definitions/README.md).

## Where will service API definitions live?

Service API definitions will live in [api-definitions](../../api-definitions) within this repo.