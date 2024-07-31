# Dependency Management Policy for InstructLab

This document describes the policies for adding and updating build and runtime dependencies of all InstructLab components.

## Best practices

<!-- This section modified from https://github.com/instructlab/training/issues/34#issuecomment-2176071856 -->
1. Express dependencies by setting a minimum version (using `>=`) to ensure compatibility.
2. Do not "pin" a dependency to a single version (using `==`).
3. Exclude specific versions known to not work (using `!=`).
4. Only apply "caps" to dependencies (using `<`) when that dependency has established a pattern of producing new releases with breaking changes.
5. Pin versions in CI, with frequent automated updates.

Best practices for python dependencies call for using ranges in package requirements and pinning versions only in CI jobs.

Using pinned versions in a list of constraints used for tests allows us to know and advertise exactly what versions have been tested in CI.
That information is useful for users and re-packagers to understand which versions of dependencies are compatible with more specificity than the ranges provide.
Tools like Dependabot will submit PRs to automatically update those pins to help us keep up with new releases of all of our dependencies.

Pinning to specific versions in the package dependencies so that installing a built package requires an exact version of a dependency is not a good practice.
It makes it very easy for sets of packages that need to be installed together to have incompatible dependencies, which in turn makes it impossible to actually install them (for example, [instructlab/training #34](https://github.com/instructlab/training/issues/34).
Pinned dependencies also make it difficult to deal with CVEs or other critical bugs in those dependencies, which makes delivering products from this project more challenging.
Do not pin to specific versions of libraries.

Instead of pinning, we use version ranges.
This ensures that re-packagers and installers have some flexibility in case a dependency of our package has a critical CVE and needs to be updated.
Those ranges should include a minimum version, and in some cases a maximum version (a "cap").

Specifying the minimum value for the range (`foo>=x.y`) allows us to declare that we need features that only show up in or after that version of the dependency, which means we won't get bugs from users trying to use instructlab with an old dependency that has an incompatible API or is completely lacking a feature we need.

Specifying a maximum value for the range (`foo>=x.y,<x+1.0`) allows us to prevent a library that is known to introduce breaking changes in new releases from being brought into dev and test environments without being tested.
That maximum version setting should be used with care, though, and only when definitely needed, because it can prevent good new versions from being used and can cause incompatibility issues similar to pinning versions.

Before setting a maximum version, wait for something to break, then set a cap to exclude the version that causes the breakage until the dependency is fixed or our code is updated to work with the new version of the dependency.
Bad releases of dependencies (with incompatibilities or known bugs) can be excluded using the requirements syntax `!=a.b`.
If the new dependency requires incompatible changes in our code, then it becomes the new minimum version for our requirements range.

## Coordinating dependencies with downstream builders

As the maintainers of InstructLab, it is our responsibility to ensure that the minimum versions of all of our dependencies are specified accurately so that downstream builders can know what to build.
There are many reasons why a new version of a dependency available on [PyPI](https://pypi.org/) might not be built downstream, including timing, source availability, build dependencies, and CVEs.
We cannot, therefore, assume that downstream builds reproduce the same set of packages that we get when we install using pip and PyPI.
If we determine that we need a new version of a dependency, we _must_ update the minimum version specifier in our requirements list to signal that need.
Problems with dependencies that are reported against downstream builds that include versions that match the upstream requirements specifications will be treated as bugs in those requirements specifications.
