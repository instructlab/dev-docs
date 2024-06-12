# API Definitions Guidelines

This document describes how service APIs will be managed for `InstructLab` and the sub-components of the `InstructLab` backend.

## What parts of InstructLab need service APIs?

There are two primary classes of service APIs needed to support InstructLab:

* `Platform APIs`: These are APIs that are ignorant of `InstructLab` and provide generic AI platform capabilities (e.g., Fine Tuning, SDG, Eval)
* `InstructLab APIs`: These are the APIs that reflect the user-facing functionality of `InstructLab` itself. They are aware of the end-to-end `InstructLab` workflow.

The `InstructLab APIs` are essential for hosting `InstructLab` as a service in a repeatable way. The `Platform APIs` are critical for component reuse and extensibility (e.g., new SDG algorithms for new taxonomy data types), but they are not strictly required for hosting `InstructLab` as a service.

## How will service APIs be defined?

Service APIs will be defined using [OpenAPI](https://www.openapis.org/) format in [YAML](https://yaml.org/).

## Where will service API definitions live?

Service API definitions will live a new repository github.com/instructlab/openapi. This repo will have two primary responsibilities:

1. House the static service API definitions
2. Build and publish any language-specific generated packages for consumption by service implementation projects (see below)

## How will service implementations reference shared APIs?

When a project chooses to implement one or more service APIs, there are three acceptable methods for doing so, listed in order of preference:

1. Consume a supported language-specific package. The `openapi` repo will build/publish/tag consumable packages with generated code for supported languages based on standard package distribution channels for the given language. This is the preferred method of consumption as it avoids repository references and code duplication.
2. For languages without a supported package, the `openapi` repo may be held as a [git submodule](https://www.git-scm.com/book/en/v2/Git-Tools-Submodules).

## Style Guidelines

* Use `kebab-case` for path elements
  * All characters must be in the [ascii](https://www.ascii-code.com/) character set to avoid percent encoding in URIs
  * All letters must be lowercase
  * Words are separated by the `-` (dash) character
* Use `snake_case` for properties
  * All characters must be in the [utf-8](https://www.w3schools.com/charsets/ref_html_utf8.asp) character set for simple `json` encoding
  * Words are separated by the `_` (underscore) character
* Use `UpperCamelCase` for internal reusable schema names
  * These are internal names, so the character set is not limited
  * Words are capitalized and concatenated with no separator

## API Layout

* There will be two main portions of the APIs:
  * `instructlab.yaml`: This defines the user-facing `InstructLab` REST API
  * `platform.yaml`: This defines the platform-level APIs used by the `InstructLab` workflow.
* Each platform `Capability` should own its own fully-functional sub-API file that can be used by individual capability service implementations
* Any schema object that is reused between endpoints should be housed in a schema file under the central `common` directory.

## Versioning and Stability

**WARNING** At this stage in development, we make no guarantees about stability and support for APIs!

**FUTURE**: Once stabilized, the APIs will follow an agreed-upon form of [semantic versioning](https://semver.org/) so that users can rely on the API's stability. The decision of how to version the API and at what granularity to do so is still under discussion.