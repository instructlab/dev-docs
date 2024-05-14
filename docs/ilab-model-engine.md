# Redesign `ilab` Command Structure to be Resource Based

This document describes a new structure for `ilab`, consisting of sub-commands that act as parent or "group" commands for further sub-commands. Currently the only group command is `ilab`. If the models `ilab` produces are meant to be the golden standard for open source AI, the tool to use and manage these models needs to be fully-fledged and extensible, especially for models we have not produced but should be able to run.

open source container tools like [podman](https://podman.io/) and [docker](https://www.docker.com/) are commonly viewed as engines. `ilab` should be a model engine, managing the deployment and creation of AI models. To be clear, I will be using tools like podman as a structural analogy here with full understanding they have different implications and use-cases.

## Key Component

### InstructLab Structure Redesign

Here are two outlines. They represent the state of `ilab` before/after this enhancement.

```console
ilab
|
|_____chat
|_____check
|_____convert
|_____download
|_____generate
|_____init
|_____list
|_____serve
|_____test
|_____train
```

vs. after:

```console
ilab
|
|_______model
|       |
|       |____convert
|       |____download
|       |____train
|       |____serve
|       |____chat
|       |____evaluate
|       |
|_______data
|       |
|       |____generate
|       |
|_______config
|       |
|       |____init
|       |
|_______taxonomy
|       |
|       |____diff
|       |____check
|       |____download
```

The main point of this structure is to introduce a resource based hierarchy and to improve the usability of the existing `ilab` structure. At the top level of commands, we are not thinking about verbs but resources commonly managed in `ilab`, under them fall some actions. Some key things missing in `ilab` currently are:

1. Ambiguity in "what" we are generating, training etc.
    - The current structure of `ilab` requires an in-depth knowledge of the ordering of commands and background knowledge for how our CLI is built. This is not the best way to grow. Commands that are not intuitive and require a deep dive into the docs just to get started might not engage users as we hope to. Having an intuitive structure in which people can see easily from the docstring what each command does and how they might use them, makes more sense. We are generating data, then training a model, running a server to interact with that model, then finally chatting with the model
2. Some commands like `ilab test` and `ilab list` should be renamed to show their true purpose (`ilab model evaluate` and `ilab taxonomy diff`)
3. We will hit a point where we want to add more commands to `ilab` due to expected integration with the backend, model registries etc. This list will grow to a point where this form of top group organization is not maintainable.

In general, moving to this structure allows for more growth when `ilab` reaches a stable release and encourages broader usage of the tool for all sorts of open-source AI models.

#### Goals for 5/30/2024 milestone release of CLI

For the next few milestones, it has been identified that the overall structure shift should be in place. We should aim to have the base hierarchy of commands in place that will exist for the future of `ilab` so that when users become familiar with the CLI, further drastic changes are not needed.

This means we will now have commands that have a parent group `ilab` followed by a sub-command/group terminated by a grandchild command: `ilab model list` for example. This workflow not only adds purposeful organization to this project, but will encourage contributors to add commands as they use `ilab`, and file bugs in specific parts of the codebase that need work as opposed to the current flow which is hard to keep track of from a user's point of view.

### Necessity of a input -> configuration -> result mental model

The commands uncovered by adding this structure resemble the typical architecture for an "engine". `ilab` is not complete without the mechanisms to create, list, delete, and inspect the models. Models, as container images do in container engines, act as the configuration for the end result: the chat process. The interesting thing about this analogy, is that there needs to be a pre-cursor to configuration as well. There is the container image, the container, and the `Containerfile`. The `Containerfile` is the raw user input that leads to an image.

`ilab` needs to have a clear source of information, a result of compiling this source, and a "running" end result. Having these three steps purposefully delineated creates the need for commands in which users to manipulate and act upon each of these step.

The mental model a user has impacts the way they use the application and whether they view it as a Proof Of Concept.

Here is a diagram of what I mean when I describe this mental model:

```console
SOURCE                                                                                                                                                                                                                                              SINK

        User has un-commited changes in their taxonomy                          Data is generated and then the user runs                               A model is created, and the user can tag the model               After converting the model, the
        or specifies `--skill` `--knowledge` in the                 ==>        `ilab model train` using the newly generated data       ==>             or just list it using the new model commands            ==>      user runs the chat procces by specifying `ilab model serve` followed by `ilab model chat
        new `ilab data generate`

```

So in `ilab` the clear source of information is the taxonomy, or the new skills/knowledge brought into generate. This data then goes through processes that end up in the sink, which is a running/actionable process which the user can interact with. Providing commands that clearly display these different processes is key. Groups for these larger "buckets" of processes make sense as well due to the likelihood of more commands being needed to properly interact with the newly generated model.

### Alternatives

An alternative flow to `ilab` -> `child-command` -> `grandchild-command` is:

`ilab` -> `verb` -> `noun`. The positive to this approach is commands would sound better: `ilab generate data`. However there are a few negatives. While this is easier to say, it makes less sense from an organizational standpoint:

```console
ilab
|
|_______download__|
|_______convert___|-----model
|_______train_____|
```

This structure works when commands are ALL linked to the same `verb`. What happens when a group like `download` gets a command that the others don't? Then it looks more like this:

```console
ilab
|
|_______download
|       |____model
|       |____taxonomy
|_______convert
|       |_____model
|_______train
|       |_____model
```

This is opposed to the structure in this EP, which might have a duplicate here and there for something like `download`, but in this structure duplicate commands are the norm and will result in a larger and clunkier codebase that is confusing to read.

This first structure looks nice, and `ilab download model` sounds nice. However in terms of implementation, this makes little sense. In [click](https://click.palletsprojects.com/en/8.1.x/), the CLI library we use, `ilab` is a "group" that commands are currently "grouped under". In this Model, `ilab` would be a group, `download`, `convert` etc would be a sub-group, and `model` would be a command under ALL of these groups. This would require an implementation of the `model` function in different packages all linking to a different group parent. From the perspective of user contributions and general code readability doesn't make much sense. Part of this design is to encourage user contributions by making the structure of the codebase logical. While these commands make the commands sound better, they make them harder to group, and understand.

### General workflow as compared to alternatives

currently one has to:

1. `ilab init`
2. `ilab download`
3. `ilab serve`
4. `ilab generate`
5. `ilab train`
6. `ilab convert`
7. `ilab serve --model XXX`
8. `ilab chat`

how to use `ilab` with this new structure:

1. `ilab config init`
2. `ilab model download`
3. `ilab data generate`
4. `ilab model train`
5. `ilab model serve`
6. `ilab model chat`

It is now clear what is happening. For example, one may ask: "What does `ilab init` do?" while `ilab config init` clearly initializes the CLI's config. A more clear example is `ilab generate`. Are we generating a model? A config? No, we are generating *data*: `ilab data generate`.

## Changes to Existing flow

The current `ilab` commands will still work. Users will be able to type commands like `ilab model train` or `ilab train` for the foreseeable future to ensure feature parity. Eventually, this alias should be removed, and only the sub-commands should probably exist.

## Known Issues

### click

click doesn't like the setup we currently have in `ilab`. So, adding sub-parent commands won't be as easy as creating a new `click.group`. We will need to make different libraries, each of which is a click group most likely.
