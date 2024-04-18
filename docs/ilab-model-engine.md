# Redesign ilab to be a Full Fledged Model Engine

This documents describes a new structure for ilab, consisting of sub-commands that act as parent or "group" commands. Currently the only group command is `ilab`. If the models `ilab` produces are meant to be the golden standard for open source AI, the tool to use and manage these models needs to be full fledged and extensible, especially to models we have not produced but should be able to run.

open source container tools like podman an docker are commonly viewed as engines. `ilab` should be a model engine, managing the deployment and creation of AI models. To be clear, I will be using tools like podman as a structural analogy here with full understanding they have different implications and use-cases.


## Key Component

### InstructLab Structure Redesign

Here are two outlines. One that represents the state of `ilab` after this enhancement and the second describes `ilab` after I introduce new key commands to this structure. I am including the second outline for clarity purposes, but this EP only covers the contents of the first structure.

```console
ilab
|
|_______model
|       |
|       |____convert
|       |____download
|       |____train (--convert)
|       |____run   (-i)
|       |____chat
|       |____inference 
|       |
|_______data
|       |____generate
|       |
|_______config
|       |
|       |____init
|       |
|_______tax
|       |
|       |____diff
|       |____check
|       |____download
```

vs. currently:

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

The main point of this structure is to introduce a group hierarchy and to remove some confusions of the existing `ilab` structure. Some key things missing in `ilab` currently are:

1. `ilab init` does too many things.
    - We don't want "black box" commands. This creates bad UX and also ties too many processes together in which one might eventually need to be changed.
2. ambiguity in "what" we are generating, training etc.
    - The current structure of `ilab` requires an in depth knowledge of the ordering of commands and background knowledge for how our CLI is built. This is not the best way to grow. Commands that are not intuitive and require a deep dive into the docs just to get started might not engage users as we hope to. having an intuitive structure in which people can see easily from the docstring what each commands does and how they might use them, makes more sense. We are generating data, then training a model, running a server to interact with that model, then finally chatting with the model
3. some commands like `ilab test` and `ilab list` should be renamed to show their true purpose (`ilab model inference`and `ilab tax diff`)
4. we will hit a point where we want to add more commands to `ilab` due to expected integration with the backend, model registries etc. This list will grow to a point where this form of top group organization is not maintainable.

In general, moving to this structure allows for more growth when `ilab` launches publicly and encourages broader usage of the tool for all sorts of open source AI models.

Outside the scope of this EP, but still important in the near future, the following structure is preferred:

```console
ilab
|
|_______model
|       |
|       |____quantize *
|       |____convert
|       |____download
|       |____train (--convert)
|       |____top *
|       |____run   (-i)
|       |____chat
|       |____inference
|       |____push *
|       |____rm *
|       |____ls *
|       |____tag *
|       |
|_______data
|       |____generate
|       |____mix *
|       |
|_______config
|       |
|       |____create *
|       |____edit *
|       |____rm *
|       |
|_______tax
|       |
|       |____diff
|       |____check
|       |____download
```

The starred commands are new, and will be dealt with in another enhancement.

#### Goals for 4/25 Milestone in CLI

For the next few milestones, it has been identified that the overall structure shift should be in place. We should aim to have the vauge hierarchy of commands in place that will exist for the future of `ilab` so that when users become familiar with the CLI, it does not change radically soon after launch.

The primary focus here should be the `ilab model` group. The other ones should not be treated as blockers for launch, but would be nice to have.

Another key change that should be included is a switch to using `/var/lib/models` as a model store and `$HOME/.config/models/ilab.conf` for the new configuration path. Keeping config and models in the CWD does not map well to the importance of these two pieces. This will allow `ilab config` commands to act on a global level per-user. Models will be a system-wide directory as they are with most other applications of this type.

This means we will now have commands that have 2 nouns followed by a verb: `ilab model list` for example. This workflow not only adds purposeful organization to this project, but will encourge contributors to add commands as they use `ilab`, and file bugs in specific parts of the codebase that need work as opposed to the current flow which is hard to keep track of from a user's point of view.

### Necessity of a input -> configuration -> result mental model

The commands uncovered by adding this structure resemble the typical architecture for an "engine". `ilab` is not complete without the mechanisms to create, list, delete, and inspect the models. Models, as container images do in container engines, act as the configuration for the end result: the chat process. The interesting thing about this analogy, is that there needs to be a pre-cursor to configuration as well. There is the container image, the container, and the Containerfile. The Containerfile is the raw user input that leads to an image. 

`ilab` needs to have a clear source of information, a result of compiling this source, and a "running" end result. Having these three steps purposefully delineated creates the need for commands in which users to manipulate and act upon each of these step.

The mental model a user has impacts the way they use the application and whether they view it as a Proof Of Concept.

Here is a diagram of what I mean when I describe this mental model:

```console
SOURCE                                                                                                                                                                                                                                              SINK

        User has un-commited changes in their taxonomy                          Data is generated and then the user runs                               A model is created, and the user can tag the model               After converting the model, the
        or specifies `--skill` `--knowledge` in the                 ==>        `ilab model train` using the newly generated data       ==>             or just list it using the new model commands            ==>      user runs the chat procces by specifying `ilab  model run -i` or `ilab model run` followed by `ilab model chat
        new `ilab data generate`

```

So in `ilab` the clear source of information is taxonomy, or the new skills/knowledge brought into generate. This data then goes through processes that end up in the sink, which is a running/actionable process which the user can interact with. Providing commands that clearly display these different process is key. Addings groups for these larger "buckets" of processes makes sense as well due to the likelihood of more commands being needed to properly interact with the newly generated model.


## Changes to Existing flow

The current `ilab` commands will still work. Users will be able to type commands like `ilab model train` or `ilab train` for the forseeable future to ensure feature parity. Eventually, this alias should be removed, and only the sub-commands should probably exist.

## Known Issues

### click

click doesn't like the setup we currently have in `ilab`. So, adding sub-parent commands won't be as easy as creating a new `click.group`. We will need to make different libraries, each of which is a click group most likely. 
