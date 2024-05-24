# Confluence document source

Importing information from Confluence is crucial for fine-tuning models on internal documentation.
Many companies use Confluence to store their internal documents.
Fine-tuned models can be employed within these companies and shared externally without compromising the internal documentation itself.
Therefore, importing information from Confluence benefits both companies and the broader community.

## Interfaces

qna.yaml file, `document` section:

- Confluence Host: The base URL of the Confluence instance.
- Space: The Confluence space key where the documents reside.
- Page titles: The titles of the Confluence pages to fetch.
- Version: The version of the Confluence page.

The qna.yaml file can define single host and multiple spaces and pages,
each with an optional version.

Confluence credentials in config.yaml:
- Username
- [Token](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)

## Changes across modules

- [Configuration module](https://github.com/instructlab/instructlab/blob/main/src/instructlab/config/config.py)
  defines the structure and validation rules for
  the config.yaml file.
- [Schema module](https://github.com/instructlab/schema) defines the structure and validation rules for
  the qna.yaml file.
- [sdg utilities module](https://github.com/instructlab/sdg/blob/main/src/instructlab/sdg/utils/taxonomy.py)
  fetches documents
- [unit test](https://github.com/instructlab/sdg/tree/main/tests)

## Additional External Packages

The implementation relies on the following external packages:

- [atlassian-python-api](https://atlassian-python-api.readthedocs.io/) –
  A Python library to interact with Atlassian products, including Confluence.
- [markdownify](https://pypi.org/project/markdownify/) –
  A library to convert HTML content to Markdown for processing Confluence page content.
