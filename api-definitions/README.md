# API Definitions

The common abstract interface for the service APIs is defined by [REST APIs](https://www.redhat.com/en/topics/api/what-is-a-rest-api) using [OpenAPI](https://www.openapis.org/). This project is the central home for these API definitions.

## Style Guidelines

* Use `kebab-case` for path elements
* Use `snake_case` for properties
* Use `UpperCamelCase` for internal reusable schema names

## API Layout

* There are two main portions of the APIs:
  * [instructlab.yaml](./instructlab.yaml): This defines the user-facing `InstructLab` REST API
  * [platform.yaml](./platform.yaml): This defines the platform-level APIs used by the `InstructLab` workflow.
* Each platform `Capability` should own its own fully-functional sub-API file that can be used by individual capability service implementations
* Any schema object that is reused between endpoints should be housed in a schema file under [common](./common)
