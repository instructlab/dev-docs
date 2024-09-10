# `docs.instructlab.ai` website

## Problem statement

Reading the numerous `README.md` files for the InstructLab Project has become overwhelming and cumbersome.
Understanding the branching structure of the different ways to implement and use InstructLab is not only
confusing but frustrating to beginning users. Not having a centralized documentation site that is the
first and default stop for our downstream users is doing us a disservice.

## Proposed resolution

1) Leveraging the static site ability of GitHub we spin up the <https://github.com/instructlab/docs> repository.
and create a `CNAME` to point to <https://docs.instructlab.ai>.
2) We migrate the different `README.md`s to this static site, (most likely [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/),
and iterate organizing the documentation via PRs and useful flow of information.
3) We leverage our TechWriters to have a single location to update official "downstream" documentation.
4) We have the dedicated `README.md`s for each project, but stripped down to very specific niche things for the sub-project, or fast feedback development notes.
