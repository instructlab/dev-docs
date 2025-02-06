---
author: Laura Santamaria (@nimbinatus)
date: 05 February 2025
status: proposed
---

## Issues

Our taxonomy tree structure and knowledge/skill file structure was designed with upstream taxonomy submissions in mind. An end user working with a taxonomy locally using InstructLab has to follow all of those requirements, increasing complexity of their work.

The end user typically gets hung up on where to place the file in a massive file tree where sub-branches are not defined. For someone not working in the upstream taxonomy, this is basically bikeshedding[^1]. The only requirement for the SDG process is sorting things into `knowledge` and `skills`.

The user experience of working with the `qna.yaml` file is poor for a handful of reasons:

- Most of the fields in the `qna.yaml` file are unnecessary save for use in the upstream taxonomy.
- YAML is a notoriously complex, loose format with a lot of potholes.
  - YAML files of different specifications parse completely differently (e.g., 1.2 vs 1.1).
    - Note that PyYAML, our base tool [^2], parses YAML 1.1, not 1.2. There is a long way to go[^3] to support 1.2, which has been the latest spec since 2009. As such, even if someone were to search the Internet for a solution because they are not familiar with YAML, they likely will stumble across 1.2 solutions that don't work for 1.1.
  - There are at least 9 different ways to indicate a multi-line string in YAML[^4], depending on which block scalar indicator[^5] is used and which block chomping indicator[^6] is used (this does **not** count the indentation indicator[^7]!). Then there are double-quoted flow scalar multilines[^8] and single-quoted flow scalar multilines[^9], which can cause more problems.
- The linting system, intended to ensure the YAML file is readable by the SDG process, adds more burden on the non-technical user.
  - The linter for YAML enforces an 80-character line length by default. That makes sense if you're working on code read from a terminal, but not to a typical end user used to working with rich text editors for a reading comprehension experience working with paragraphs.
  - The linter also complains about trailing whitespace, another common thing that the typical end user won't understand why everything is failing.

From a code perspective,

- We are already using JSON in the data mixing process in SDG[^10].
- Docling also exports JSON as input and output[^11].
- JSON is also much more friendly to UI work, which is a primary path we would like people to use.

Overall, the `qna.yaml` file needs to have fewer knobs and fewer pitfalls.

The process of writing question and answer sets also is more like writing reading comprehension sets from a standardized exam. It would be better to frame this hands-on part of the process as similar to the passage and question sets from English reading comprehension exams

## Proposed solutions

To fix the user experience when working with data, I propose the following ideas. In general, the basic idea is "Keep It Simple; Make It Tick."

### Use a schema field rather than directory tree structure

Drop the folder structure in favor of a schema field for submission type and even domain, if necessary. The schema field can be entered automatically via the UI through a user selecting `knowledge` or `skill`.

### Streamline the schema

Make `created_by`, `domain`, and `document_outline` optional fields. Enforce those fields for the upstream taxonomy though documentation, CI, and review rather than require them for everyone.

### Switch to JSON and Markdown for the `qna.yaml` document

Allow the user to use Markdown in a WYSIWYG experience, and then use a Markdown-to-JSON converter to handle the conversion to a code-friendly format.

Markdown is very user-friendly, and converters handle a lot of the issues with encoding and special characters that happen in situations like working in other languages. We don't have to worry about a linter arguing about line length with the end user, and we wouldn't have to think about whether the user used tabs or spaces or forgot to strip whitespace at the end of a line.

This would also make it a lot easier for the UI to work with contributions. JSON plays well with JavaScript overall without importing more libraries and creating dependency issues, and Python has a very good built-in for working with JSON files.

Users who decide to build it without needing the converter are likely familiar with JSON, and there are fewer pitfalls and less likelihood of tooling choices impacting meaning as the JSON standard has not changed since 2017, and barely changed from the original standard.

### Reframe the Q&A writing process as a reading comprehension process

Write documentation and tutorials based on existing tutorials on writing reading comprehension questions and example answers for standardized exams.

Most people can understand reading to learn versus learning to read type questions. The new, streamlined schema that matches the most simple needs could help here along with a solid set of docs and tutorials on how to write reading comprehension sets. We could borrow heavily from the standard tutorials for writing standardized exams that are out there for free and already battle-tested.

## Unaddressed concerns

The issue of needing a git repository for document storage is possibly out of scope of this document. However, I'm adding it as something that may need its own ADR/dev doc. The end user experience of needing a git repository is needlessly complex and also still follows the idea of the upstream taxonomy and community model build. A user working with InstructLab locally does not need the version tracking provided by git and likely probably already has a document storage system. I propose changing the general idea from a git repository to a simple address, whether that's local storage, remote storage, or a version-controlled repository. Make it more flexible.

∎

[^1]: The story of the bikeshed is a common metaphor. The story goes that a group that is working on the approvals for the construction plan of a nuclear power plant gets stuck on what color to paint the bike shed at one of the entrances to the plant. Multiple meetings are scheduled to hash out the issue of the color of the bike shed, with heated arguments. However, the rest of the plan for the power plant is not examined in detail or critiqued. People have an easier time evaluating and having an opinion on something that is as trivial as a bike shed's color when faced with complex decisions on other systems. [Wiktionary entry](https://en.wiktionary.org/wiki/bikeshedding)
[^2]: [Our tooling dependencies](https://github.com/instructlab/schema/blob/main/pyproject.toml#L27-L30)
[^3]: [yaml/pyyaml#486](https://github.com/yaml/pyyaml/issues/486)
[^4]: You can experience this issue in action with the interactive experience on [yaml-multiline.info](https://yaml-multiline.info/).
[^5]: [YAML Spec v1.2.2 on block scalar styles](https://yaml.org/spec/1.2.2/#81-block-scalar-styles)
    > YAML provides two block scalar styles, literal and folded. Each provides a different trade-off between readability and expressive power.
[^6]: [YAML Spec v1.2.2 on block chomping indicators](https://yaml.org/spec/1.2.2/#8112-block-chomping-indicator)
    > Chomping controls how final line breaks and trailing empty lines are interpreted. YAML provides three chomping methods:
[^7]: [YAML Spec v1.2.2 on block indentation indicators](https://yaml.org/spec/1.2.2/#8111-block-indentation-indicator)
    > Every block scalar has a content indentation level. The content of the block scalar excludes a number of leading spaces on each line up to the content indentation level.
    >
    > If a block scalar has an indentation indicator, then the content indentation level of the block scalar is equal to the indentation level of the block scalar plus the integer value of the indentation indicator character.
    >
    > If no indentation indicator is given, then the content indentation level is equal to the number of leading spaces on the first non-empty line of the contents. If there is no non-empty line then the content indentation level is equal to the number of spaces on the longest line.
    >
    >It is an error if any non-empty line does not begin with a number of spaces greater than or equal to the content indentation level.
    >
    >It is an error for any of the leading empty lines to contain more spaces than the first non-empty line.
    >
    >A YAML processor should only emit an explicit indentation indicator for cases where detection will fail.
[^8]: [YAML Spec v1.2.2 on the double-quoted flow scalar](https://yaml.org/spec/1.2.2/#double-quoted-style)
    > In a multi-line double-quoted scalar, line breaks are subject to flow line folding, which discards any trailing white space characters. It is also possible to escape the line break character. In this case, the escaped line break is excluded from the content and any trailing white space characters that precede the escaped line break are preserved. Combined with the ability to escape white space characters, this allows double-quoted lines to be broken at arbitrary positions.
[^9]: [YAML Spec v1.2.2 on the single-quoted flow scalar](https://yaml.org/spec/1.2.2/#single-quoted-style)
    > In addition, it is only possible to break a long single-quoted line where a space character is surrounded by non-spaces. [...] All leading and trailing white space characters are excluded from the content. Each continuation line must therefore contain at least one non-space character. Empty lines, if any, are consumed as part of the line folding.
[^10]: stuff
[^11]: [The Docling documentation](https://ds4sd.github.io/docling/supported_formats/) notes docling supports JSON-serialized Docling Documents and Markdown as input and JSON and Markdown as outputs.
