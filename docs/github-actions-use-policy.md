# GitHub Actions Use Policy for InstructLab

This document describes the use policy for GitHub Actions (actions) in workflows for all repositories in the [InstructLab](https://github.com/instructlab) organization.

## Background

GitHub Actions are an important tool for CI/CD use within the repositories of the InstructLab project.
One of the main values is to verify the quality of pull requests for things like tests passing, spelling checks, well-formedness of files, etc.
Repositories may also use actions to build and publish releases for the project.

Since actions play a critical role in the project, care must be taken in how they are used due to their place in the security of the software supply chain of the project.

## Dependabot

Each repository using GitHub Actions must configure Dependabot to manage the action dependencies.
The repository must contain a `.github/dependabot.yml` file with the following minimum configuration:

```yaml
version: 2
updates:
  # Maintain dependencies for GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "daily"
```

Repository maintainers must review and respond to all pull requests submitted by Dependabot in a timely manner.

## SHAs instead of tags

GitHub Actions must be referenced using SHA values for a specific commit.
Use of other reference types such as tag or branch names is not permitted.

```yaml
- name: Git Checkout
  uses: actions/checkout@0ad4b8fadaa221de15dcec353f45205ec38ea70b # v4.1.4
```

instead of

```yaml
- name: Git Checkout
  uses: actions/checkout@v4
```

The use of SHAs can avoid security issues if the tag or branch of the action is moved to a malicious commit.
We also gain build repeatability for future builds by referring to a precise revision of the action.

Since we use Dependabot to manage our action dependencies, Dependabot will handle the chore of using the proper SHA values in the submitted pull requests when action dependencies are updated.

## Trusted Providers of GitHub Actions

There are many GitHub Actions available in GitHub.
Not all can be necessarily trusted.
The InstructLab project must maintain [a list of allowed providers and a list of denied providers](github-actions-providers.md).

Allowed providers will include all GitHub created actions (`actions/*`) as well as other official actions such as Python Packaging actions (`pypa/*`).

The InstructLab organization's Settings->Actions->General must be configured to allow select actions including actions created by GitHub along with the allowed providers.

Adding actions to the allowed providers or denied providers lists will require approval by the organization maintainers along with updating the organization's settings. This can be done by submitting a Pull Request to modify [`github-actions-providers.md`](github-actions-providers.md).
