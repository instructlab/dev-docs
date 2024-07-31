
# DeepSearch + InstructLab Integration Proposal

<https://github.com/DS4SD>

## Why is a Conversion Tool Necessary?

Managing submissions for the open-source InstructLab project has revealed a significant bottleneck in processing
knowledge documents. For the InstructLab backend to effectively utilize these documents, they must be in markdown
format. Currently, we only accept Wikipedia articles, but the built-in conversion tool is inadequate. Internally at
IBM, and other companies, many knowledge submissions are in multiple document formats, including PDF format,
necessitating conversion to markdown before being used in InstructLab.

Existing open-source methods, such as PanDoc, are inconsistent. While they preserve text, they struggle with parsing
tables and special symbols, as evidenced by issues in PR #1154 of the taxonomy repo in the InstructLab project. Other
open-source solutions have similar shortcomings.

## Why DeepSearch?

IBM's DeepSearch software excels in document conversion, outperforming traditional open-source methods. Utilizing a
computer vision model layer, it accurately parses content in the files, including titles, headers, and tables.
Additionally, it automatically implements RAG layers for models, which could benefit the InstructLab process in
the future.

## Integration Proposal

To maintain the open-source nature of the project while leveraging the strengths of DeepSearch, we propose a
two-pronged approach:

### Open-Source Conversion

- Implement a basic document conversion tool in the UI using an open-source method such as PanDoc. This tool will be
lightweight and easily hosted, ensuring it can be used and improved by the community.

### DeepSearch Integration

- Enable the UI to switch the conversion endpoint to DeepSearch, allowing high-fidelity markdown conversions for
backend use. This approach maintains an open-source version while benefiting from DeepSearch's superior
conversion capabilities.

IBM Research and the DeepSearch team will host the DeepSearch endpoint for the open-source community. This
arrangement benefits the community by streamlining contributions and provides data and exposure for the DeepSearch
project. IBM's contribution underscores its commitment to supporting and improving open-source projects.

This integration will highlight the value of DeepSearch, highlighting their potential for those integrating
InstructLab into their workflows. If the volume of community requests becomes unsustainable for the DeepSearch team,
we hope for ample notification to allow the community to find alternative solutions. By then, we anticipate that the
open-source versions will have improved sufficiently, or the value of the integration will justify continued support.

By adopting this two-pronged approach, we ensure the integrity of the open-source project while leveraging IBM's
advanced DeepSearch capabilities. This strategy balances community collaboration with innovative technology,
fostering innovation and improvement in document processing for the InstructLab project.
