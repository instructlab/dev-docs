# Knowledge Document Ingestion Pipeline Design Proposal

**Author**: Aakanksha Duggal

## 1. Introduction

As part of extending InstructLab's capabilities, this pipeline is designed to support ingestion and processing of various document formats such as Markdown, PDF, DOCX, and more. The goal is to create a unified, modular system that seamlessly integrates with **Synthetic Data Generation (SDG)** and **Retrieval-Augmented Generation (RAG)** workflows, simplifying the process for users while maintaining flexibility for future enhancements.

## 2. Use Case

To enable the ingestion and processing of a wide range of document types, this pipeline must handle formats including:

- Markdown (MD)
- PDF
- TXT
- ASCII Docs
- DOCX
- HTML
- PPTX

This proposal outlines how to build a pluggable system that accommodates these formats and integrates effectively into existing **SDG** and **RAG** workflows.

## 3. Proposed Approach

### 3.1 Custom InstructLab Schema Design

- **Objective**: Define a custom "instructlab" schema that standardizes input formats for SDG and RAG pipelines. This schema will bridge the gap between various document types and the specific formats required for further processing.
- **Modularity**: The system should allow easy extension, enabling support for new document types and processing workflows without disrupting core functionality.
  
The instructlab schema will serve as the intermediary format, supporting flexibility while ensuring compatibility with the ingestion process.

### 3.2 PDF and Document Conversion via Docling

- **Docling Integration**: We will leverage **Docling** to convert files into structured JSON, which will be the starting point for the instructlab schema. Individual components will post-process this JSON as per the requirements of the specific SDG and RAG workflows.
  
- **Docling v2**: We have collaborated with the Docling team to extend their tool’s capabilities, allowing conversion for PDF, HTML and DOCX formats. These new document types will be supported in the ingestion pipeline via docling's upgraded v2 release.

### 3.3 Introducing the Document Chunking Command

- **Command Overview**: We propose a new command, `ilab document format`, which will:
  - Take a document path (defined in `qna.yaml`).
  - Format and chunk the document into the desired schema.
  
- **Implementation Details**: Initially, this functionality will be integrated into the existing SDG repository. Over time, it can evolve into a standalone utility, allowing external integrations and wider usage.

- **Example Workflow**:
  ```bash
  ilab document format --input path/to/document.pdf --output path/to/schema
  ```
  This command will ingest a document, process it into the instructlab schema, and output the result for further use in SDG or RAG workflows.

### 3.4 Simplifying Git-Based Workflows for Users

- **Current Challenge**: Knowledge documents are stored in Git-based repositories, which may be unfamiliar to many users.
- **Proposed Solution**:
  - Allow users to input a local directory and provide an automated script that:
    1. Initializes a Git repository.
    2. Creates the necessary branches.
    3. Organizes files into the required structure.
  
  By abstracting the Git setup process, we can retain Git’s benefits (version control, backups) while simplifying the interface for non-technical users.

- **Implementation Example**:
  ```bash
  ./setup_git_repo.sh --input /path/to/local/docs
  ```
  This script automates the process of structuring knowledge documents for ingestion.

### 3.5 Workflow Visualization

Here is a conceptual diagram illustrating the workflow from document ingestion to schema conversion and chunking:

![Knowledge_Document_Ingestion_Workflow](https://github.com/user-attachments/assets/06504b1b-bc8f-4909-b6a2-732a056613c5)

The pipeline begins with the ingestion of knowledge document, passes through a conversion step to instructlab schema using **Docling**, and then processes the document into a format usable by SDG and RAG workflows.

## 4. Future Enhancements

### 4.1 Support for Additional Data Types

To extend beyond text-based documents, the pipeline will explore handling other formats, including:
- **Audio and Video**: Incorporating media formats will require modifications to the schema and additional processing capabilities.
- **Visual Language Models (VLMs)**: We will collaborate with the research team to align this work with visual data processing tools and extend the pipeline to handle multimedia.

### 4.2 Refined Chunker Library

- **Standalone Library**: The document chunking functionality will eventually be refactored into a dedicated chunking library or utility within the SDG module. This separation will make it easier to maintain and extend in future iterations.
  
- **Performance Optimizations**: Ongoing work will aim to reduce the time and resources needed for large-scale document chunking, particularly for multi-format documents like PDFs containing both text and images.

## 5. InstructLab Schema Overview

### Key Components:
- **Docling JSON Output**: The output from Docling will be the instructlab schema, which serves as the backbone for both SDG and RAG workflows. For specific details around the leaf node path or timestamp, we will include that as a part of the file nomenclature.
  
This schema will standardize the data format for all supported document types, enabling consistency and modularity in the pipeline.
