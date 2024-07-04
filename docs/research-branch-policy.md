# Research Branch Policy for InstructLab

This document proposes a research branch policy for InstructLab repositories (sdg, eval, training) in the InstructLab organization.

## Background

Research engineers contributing to InstructLab have expressed the need for the ability to make rapid, experimental changes in a space where they can collaborate and move quickly without working through robust, production-level code review. 
Software engineers contributing to InstructLab have expressed concern about allowing code into the `main` branch that does not meet the production-level quality standards enforced on main. We need to minimize the time from code merged into main to shipping in a release, and automating confidence in that shipment.

One of the more unique considerations InstructLab has as a project in contrast to other open source projects is the tight collaboration between research engineers and software engineers. There are cultural differences and different best practices in each discipline, and in order to enable this partnership we may need to take some new or less standard approaches to open source project management.

### Current temporary solution

A temporary working solution to enable research engineers to make forward progress is creating a `research` branch in a fork of the InstructLab repository under a user account. This enables the researchers to move forward rapidly and experiment in a collaborative manner. This will also necessitate a more involved merge process once the branch is ready to merge into the `main` branch of the `instructlab` repo under the InstructLab GitHub organization. That is a known and accepted tradeoff of not doing continuous production-level code review.

### Proposed long-term solution

Longer-term, however, the researchers have expressed concern that having this `research` repository in a user's fork makes it more difficult for would-be research-oriented contributors to discover. They would like to have a `research` branch (akin to the common practice of having a `devel` branch) on the main `instructlab` repositories such that their work is discoverable in addition to not negatively impacting the corresponding production-quality `main` branch. 

### Concern about a specialized branch in main

The software engineers have expressed concern that having a non-standard branch like this on the main upstream repository and not in a fork is against standard upstream open source practice:

- It creates social strata between contributors who have the permissions to create a branch in the main repo vs contributors who can only create a branch in a user fork. "Official branches in the repo, all development in forks."
- PRs for both the `main` (production-quality) and `research` (rapid and experimental) will live in the same repository. This might cause confusion for PR review workflows - it may not be clear to the PR reviewer which branch the PR is destined for clearly depending on the tooling being used.

# Git branch strategies

There are different strategies around git branching:

- [Release-based branching](https://medium.com/@pooyagohardani/git-release-candidate-branches-strategy-balancing-quality-and-agility-in-software-development-21cc00842e03)
- [Feature branching](https://martinfowler.com/articles/branching-patterns.html)
- [Git flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [Task branching](https://unity.com/how-to/devops-task-branch-workflow)
- [Trunk-based development](https://trunkbaseddevelopment.com/)
- Probably others!

(More information on some of these including a comparison chart is available in [Git Branching Strategies for DevOps: Best Practices for Collaboration](https://dev.to/angelotheman/git-branching-strategies-for-devops-best-practices-for-collaboration-35l8) by Angel Oduro-Temeng Twumasi on dev.to.)

The CLI repository [instructlab/instructlab](https://github.com/instructlab/instructlab) has a [Release Strategy devdoc](https://github.com/instructlab/instructlab/blob/main/docs/release-strategy.md) that dictates a release-based branching strategy. We do not have defined branching strategies for the other repos.

We should probably decide on explicit branching policies for each of the core repositories in the `instructlab.` This will set clear expectations for our ways of working as a community project.

# Research branch policy proposal

Regardless of the branching strategy of the repo, we propose allowing a `release` branch on each of the core `instructlab`. This branch would not be subject to the same PR review guidelines as the rest of the repository. The goal of this branch is to enable rapid, multi-person collaborative development to get the code to a state that can then go through a more stringent code review process that adheres to the project's standards:

- The `research` branch in each repository that has one will have named maintainers. Maintainer membership will be based on [the InstructLab maintainer policy already in place](https://github.com/instructlab/community/blob/main/GOVERNANCE.md#project-maintainers-overview).
- Research-oriented PRs must be made against the `research` branch.
- `research` branch maintainers will have the freedom to determine their PR merge policy. It will be documented in their branch's README or another easily-discoverable location of their choosing.
- (Need to address CI in this policy. Not sure how or what is fair.)
- When the `research` branch is in a state the maintainers are confident is good enough to attempt merging to main, they will coordinate with the repository core maintainers to start a review process.
- The responsibility for completing the merge process by responding to review feedback is on the software engineer maintainers of the `research` branch and is not the responsibility of the research engineer maintainers. The research engineer maintainers will assist the software engineers responsible for the merge, however.
- The merge code review for the `research` branch will adhere to the review requirements of the repository and follow the standard PR process already documented.

The creation of this policy is in acknowledgement of the different ways of working inherent to the research engineer role in contrast to the production-level software engineer role. As a project, InstructLab has a goal of enabling upstream, open source collaboration between AI researchers, and this is one of the ways we can help achieve that goal.
