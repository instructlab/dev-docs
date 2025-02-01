# Create Separate Repo for Custom GitHub Actions

## Idea Overview

Create a separate repository within the `instructlab` GitHub org to house our custom CI Github Actions.

## Motivation for this Proposal

Within our `instructlab` GitHub org, we have an in-house GitHub action that we use across several repos: `free-disk-space`.

Examples:

- [`free-disk-space` in the `eval` repo](https://github.com/instructlab/eval/blob/main/.github/actions/free-disk-space/action.yml)
- [`free-disk-space` in the `instructlab` repo](https://github.com/instructlab/instructlab/blob/main/.github/actions/free-disk-space/action.yml)
- [`free-disk-space` in the `sdg` repo](https://github.com/instructlab/sdg/blob/a532a8d99ffe447152948e9442464923aba62637/.github/actions/free-disk-space/action.yml)

This GitHub action is universal as it is a simple script used to clean up disk space on GitHub runners and can be modified to free disk space on our CentOS-based EC2 runners.

Note that all of these in-house `free-disk-space` action files are exactly the same, so we essentially have the same file stored in three different repos.

## Pros and Cons

Below are some pros and cons of creating a separate repository to house our GitHub actions.

### Pros

- We will have one action file in one location
  - Easier to make changes in one location instead of (our present) three locations
  - Contributors will know where to look for in-house action files
- We can create releases and utilize version control

### Cons

- Extra repository to maintain.
- We can't publish any of our actions to the GitHub marketplace if we have multiple actions stored in one repository.\*

\* In reference to the last bullet point, [the GitHub docs for publishing Actions](https://docs.github.com/en/actions/sharing-automations/creating-actions/publishing-actions-in-github-marketplace#about-publishing-actions) states:
> Actions are published to GitHub Marketplace immediately and aren't reviewed by GitHub as long as they meet these requirements:
>
> - The action must be in a public repository.
> - Each repository must contain a single action.

If we do care about publishing our actions, then we should consider creating separate repositories for these actions. If we don't care to publish, then this is a non-issue. (See next section below.)

## Additional Info

Even if we cannot publish our actions to the GitHub marketplace, we can _still_ use these actions in our repository. For example, if our repo was named `ci-actions` with this layout:

```bash
.
├── custom-action-1/
│   ├── action.yml
├── custom-action-2
│   ├── action.yml
```

...then we'd reference them in our other repos like so:

```yaml
name: Some Name

on:
  workflow_dispatch:

jobs:
  some-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout
  
      - name: Use custom action 1
        uses: instructlab/ci-actions/custom-action-1@main
  
      - name: Use custom action 2
        uses: instructlab/ci-actions/custom-action-2@main
```

Reference: [StackOverflow - "How to Publish Multiple GitHub Actions from a Single Repo and Call Them from Another Repo"](https://stackoverflow.com/a/79100136)