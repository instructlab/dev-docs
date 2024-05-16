# Taxonomy Tree Organization

This document describes the justification and decision to choose
to emulate the [Wikipedia taxonomy](https://en.wikipedia.org/wiki/Wikipedia:Contents) structure for our `knowledge/` tree in the taxonomy repository.

The definition of taxonomy from Wikipedia:
> A taxonomy is a scheme of classification, especially a hierarchical classification, in which things are organized into groups or types. Among other things, a taxonomy can be used to organize and index knowledge (stored as documents, articles, videos, etc.), such as in the form of a library classification system, or a search engine taxonomy, so that users can more easily find the information they are searching for. Many taxonomies are hierarchies (and thus, have an intrinsic tree structure), but not all are.

## Why do we need this?

Our taxonomy tree is not only how humans will  place
the different `qna.yaml`s, but it's how people will look for
and update changes for specific questions and answers.
Having the challenge of this organization, copying Wikipedia's
tree is a good default standard.

## What are we going to do to enforce this?

The triage team will take into consideration the location of the
directory and how it pertains to the suggested tree that Wikipedia
publishes their as. The merging of [this PR](https://github.com/instructlab/taxonomy/pull/780)
will build the initial tree, which we can work with the backend
team to solidify the tree going forward.
Creating a new "top level" directory will require understanding
that it will be a special rare case, while lower left nodes of
the tree as long as they are put in logical place is empowered
by the contributor.
The side effect leveraging using this we can verify where the
knowledge is placed on Wikipedia and reinforce the location
in the taxonomy tree.

## Conflicts and Resolutions

With adopting this format and structure there will be conflicts and debate
about the placement of the `qna.yaml`. The triage team will do their best
to take into consideration of the challenges that may arise, and work
with the contributor to hear and engage with that conflict. The triage
team has the ultimate decision on the location of the directory and
file.
