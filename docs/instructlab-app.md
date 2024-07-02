# InstructLab macOS App

## Scope

This document is targeted for macOS applications, but the idea could easily be transferred to other operating systems.

## Problem statement

Starting InstructLab on your local laptop is hard. It requires a significant amount of `python` knowledge and terminal
work, that is unrealistic for a non technologist to use. Having to install `git` specific versions of `python` and
`xcode` requires a level of expertise that will create barriers of adoption to the InstructLab project.

## Proposed solution

[ollama][ollama] has a macOS application that is a double-click installation for their server to run the commands
locally. We propose creating the same "system bar" application, with the ability to run `ilab model serve` in the background
and a possible way to do `ilab model chat` from said application.

Having the `ilab` dog up in the system bar telling you that `ilab model serve` is running, could open up the opportunity
to have a model open to ask a quick question to the local model, and even an ability to open up a "long-running"
conversation via a web browser or the like.

## Next steps

1. Create a simple MVP of starting the `ilab model serve` application, with controls for the `serve` options, including
   what model you'd like to run, i.e. Granite or Merlinite.
2. Create an option to ask a quick question (`-qq` option) to the via the drop-down
3. Create a `ilab model chat` type interface via a window or web browser.

[ollama]: https://ollama.com/download/mac
