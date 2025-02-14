# InstructLab Custom Exceptions Pattern

## Context

The historically pervasive pattern for exception handling during application runtime is to catch internally raised exceptions in the CLI layer and use `click.secho` followed by `click.exceptions.Exit` to display a useful error message to the user before exiting the application. This leaves a risk of intermediate calls between the site of the exception and the user-facing layer changing, leading to missed new exceptions and outdated caught exceptions. A second issue is that of discoverability and verification: given a `click.exceptions.Exit` exception handling, it is not clear from the code where the caught exception originates from in the call stack, and, similarly, given a piece of code that can raise an exception, it is not clear from the local code whether that exception is properly handled in the CLI layer without investigation.

These issues will compound whenever REST APIs begin development.

## Decision

InstructLab will adopt custom exceptions to handle specific faults which are then caught in the user-facing layer (CLI, REST API, &c).

## Status

Accepted

## Consequences

* Verification of exhaustive error handling will be clear within a code section that can raise.
* CLI layer error handling will be easy to understand and trace.
* Whenever REST APIs are developed, HTTP error codes should be easier to be associated with specific exceptions.
* It should be easier to compose useful error messages for the user.
* It should be easier to correctly scope exception handling (consider a `URLError` raised about SSL verification, for example, versus a custom `SSlVerificationException`).
