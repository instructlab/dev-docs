# Wiki document source

Fetching information from wikis is an essential
feature for fine-tuning LLMs on public knowledge.

## Interfaces

qna.yaml file, `document` section:

- Wiki Host: The base URL of a wiki host.
- Page titles: The titles of the Wiki pages to fetch.
- oldid: IDs of old releases.

The qna.yaml file can define single host and multiple spaces and pages,
each with an optional version.

Example of fetch URL:

- https://en.wikipedia.org/w/index.php?title=IBM_Granite&oldid=1235007056&action=raw

Note that oldid is sufficient to reterieve a page:

- https://en.wikipedia.org/w/index.php?oldid=1235007056&action=raw

Page title is used for vaidation.

## Changes across modules

- [Schema module](https://github.com/instructlab/schema) defines the structure and validation rules for
  the qna.yaml file.
- [SDG taxonomy module](https://github.com/instructlab/sdg/blob/main/src/instructlab/sdg/utils/taxonomy.py)
  fetches documents
- [SDG unit tests](https://github.com/instructlab/sdg/tree/main/tests)

## Additional External Packages

- urllib

