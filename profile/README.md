# okforge

**Turn your scanned documents into a private, source-cited knowledge base—all on hardware you control.**

Open Knowledge Forge (**okforge**) is a local-first system that transforms scanned documents into a structured knowledge base and makes that knowledge accessible to compatible language models through the Model Context Protocol (MCP).

Most “chat with your documents” tools use a technique called RAG, or Retrieval-Augmented Generation. Think of it like giving an AI a few pages selected from a book rather than the full chapter. Those pages may contain the most relevant passages, but they can still omit important context, relationships, and supporting details—making the AI more likely to produce incomplete or unsupported answers.

**okforge** takes a different approach: it organizes your sources into a structured, interlinked digital wiki *before* you ever ask a question. It generates document summaries, maps key concepts, and extracts images—transforming raw scans into a curated library with page-level citations that let you trace generated information back to the original sources.

**Why this matters:**

* **Traceable answers:** Page-level citations let you inspect the original source and verify the AI’s interpretation for yourself.
* **Privacy and performance:** Because your data is organized in advance, capable smaller models can work with it efficiently on your own hardware. When okforge is configured with local inference, your documents and generated knowledge remain on systems you control.
* **True ownership:** Your knowledge is stored as plain Markdown files compatible with applications such as Obsidian. You are not locked into a proprietary storage format or hosted platform.

The wiki follows the current draft of the [Open Knowledge Format (OKF)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md), keeping your structured knowledge portable and readable as standard Markdown. Through okforge’s MCP server, compatible AI clients can search and retrieve that knowledge.

## The pipeline

![okforge pipeline diagram](../assets/pipeline.svg)

### How it works

1. **Extraction:** okforge uses vision-language models (VLMs) to transcribe scanned text, identify visual content, and preserve the relationship between each output and its original page. Images, diagrams, and photographs can be cropped and saved alongside the extracted text. Documents can also be translated during this stage.

2. **Structuring—the “Forge”:** Rather than producing a single long text file, okforge synthesizes the extracted material into an interlinked wiki. It identifies key concepts, creates summaries, and adds page-level citations that connect generated content to the source documents.

3. **Interaction:** Once the knowledge base is built, you can browse it as a website, search it from the command line, or connect it to an AI client through MCP. The model can locate relevant knowledge pages, read their curated content, and trace supporting information back to the original source pages.

## See a finished knowledge base

[**The Dade County Building Code of 1935**](https://okforge.github.io/dade-code-1935/) is a public knowledge base produced with **okforge** by [SRI Consultants](https://sriconsultants.net/), a Southeast Florida engineering firm that sponsored the system’s development.

The site demonstrates the output of the Forge stage: scanned documents transformed into an interlinked wiki with concept pages, full-text search, a graph view, extracted source material, and page-level citations.

For work involving historic buildings, engineers may need to consult the code under which a structure was originally permitted. SRI uses this knowledge base to research the materials and construction methods documented for historic Southeast Florida structures.

Because the source document is publicly available, it also provides an accessible example of the same local-first workflow SRI uses for private processes and project information.

## The okforge ecosystem

The system consists of two core Python packages and an optional reference Web interface. Each component can be used independently or as part of the complete document-to-knowledge pipeline.

### Core tools

* [**okforge-vision-ocr**](https://github.com/okforge/okforge-vision-ocr) — **Document extraction**

  Transcribes scanned pages into Markdown using vision-language models, extracts images and diagrams, and records the source-page mappings needed for page-level citations.

  `pip install okforge-vision-ocr`

* [**okforge**](https://github.com/okforge/okforge) — **Knowledge-base generation and access**

  Compiles extracted or existing Markdown into a structured, interlinked wiki. It also provides command-line tools and an MCP server for searching, reading, and querying the resulting knowledge base.

  `pip install okforge`

### Reference implementation

* [**okforge-webui**](https://github.com/okforge/okforge-webui) — **Local Web interface**

  Demonstrates how the core packages can be combined into a browser-based workflow with a PDF inbox, processing queue, knowledge-base browser, and publishing tools.

> [!WARNING]
> The Web UI is intended for local, single-user deployments and does not currently provide authentication or multi-user access controls. Do not expose it directly to an untrusted network.

## Knowledge in the data, not the model weights

Fine-tuning can adapt a model’s behavior, but it is not an ideal way to maintain factual knowledge that changes over time. **okforge** instead keeps domain information in structured, source-cited pages that the model reads when answering a question.

This approach allows a knowledge base to be updated without retraining a model and makes it possible to use the same content with different compatible models. Modern small and mid-sized models can work effectively with curated context, making local deployment practical for many document-analysis tasks.

Because the supporting pages retain citations to the original documents, users can trace an answer’s sources and evaluate the model’s interpretation for themselves.

## Choose the components you need

Each component can be used independently or as part of the complete pipeline:

* **Convert scanned PDFs to Markdown:** Use [`okforge-vision-ocr`](https://github.com/okforge/okforge-vision-ocr) to transcribe pages, extract visual content, and preserve source-page mappings. Its Markdown output can also be used with other document-processing or RAG systems.

* **Turn existing Markdown into a knowledge base:** Use [`okforge`](https://github.com/okforge/okforge) to organize source material into an interlinked, source-cited wiki. No OCR stage or Web interface is required.

* **Run the complete browser-based workflow:** Deploy [`okforge-webui`](https://github.com/okforge/okforge-webui) for an integrated PDF inbox, processing queue, knowledge-base browser, and publishing workflow.

## Bring your own chat client

okforge is not a chat application and does not require a vector database. Instead, it runs as an MCP server that connects your knowledge base to compatible clients such as Open WebUI, llama.cpp, Page Assist, or Hermes Agent.

This supports a hybrid workflow: a client may use its own vector search for broad semantic discovery, while okforge’s MCP tools provide full-text search and direct access to curated, page-cited knowledge pages. The model can first locate relevant pages and then read their complete contents rather than relying only on retrieved text fragments.

An okforge knowledge base represents a subject collection rather than a single source document. Multiple documents can contribute to the same concept and entity pages, allowing the knowledge base to incorporate additional sources over time. The Web UI’s MCP server can also search across all knowledge bases it hosts.

## Local-first by design

For organizations with privacy, contractual, regulatory, or data-sovereignty requirements, keeping document processing and inference under their control may be essential. okforge supports a fully self-hosted workflow in which source documents, generated knowledge bases, and model inference remain on infrastructure you manage.

Remote OpenAI-compatible inference endpoints are optional and should be used only when their data-handling practices meet your organization’s requirements.

### Tested hardware

Recent open vision-language and reasoning models—including models in the [Qwen3.6-27B](https://github.com/QwenLM/Qwen3.6) class—make a fully local scan-to-wiki pipeline practical when sufficient memory is available for the selected models and working context.

okforge is developed and used daily on systems equipped with RTX 3090, RTX 5090, and RTX 6000 Pro Blackwell GPUs. In our tested quantized configurations, a single 24 GB GPU provides a practical starting point for serial document ingestion and interactive knowledge-base access.

Actual memory requirements and performance vary with the selected models, quantization, context length, KV-cache configuration, and inference-server settings.

### Resource management and scaling

* **Serial ingestion:** Ingestion jobs are processed one at a time by default, reducing memory contention on single-GPU llama.cpp hosts.

* **No required cloud services:** A fully self-hosted deployment requires no telemetry, user account, or SaaS service in the processing path.

* **Scalable inference:** Additional VRAM and compute can support larger models, longer contexts, or multiple inference slots. Depending on the inference server and its configuration, independent read-only queries can be handled concurrently across those slots.

### Architectural flexibility

okforge separates knowledge-base management from the compute used for document processing and inference. The model server does not need to run on the same machine as okforge.

A modest server can act as a central knowledge hub for a local network while sending inference requests to any compatible OpenAI-style endpoint. That endpoint might be a GPU server elsewhere on the same network, a CPU or hybrid inference host, or an optional hosted provider.

Local endpoints keep processing within infrastructure you control. When using a hosted provider, document content and prompts may leave your systems and should be evaluated against the provider’s privacy and data-retention policies.

## Start here

Choose the path that best matches how you want to use okforge:

* **Install the core engine:** Follow the [`GETTING_STARTED`](https://github.com/okforge/okforge/blob/main/GETTING_STARTED.md) guide to create and access knowledge bases from the command line.

* **Run the complete Web interface:** Clone [`okforge-webui`](https://github.com/okforge/okforge-webui) for the integrated document-ingestion, knowledge-base management, and publishing workflow.

* **Use a coding agent:** Give the [`INSTALL_PROMPT`](https://github.com/okforge/okforge-webui/blob/main/docs/INSTALL_PROMPT.md) to a compatible coding agent for guided, interactive installation and configuration.

---

### Project heritage and credits

The **okforge** engine began as a hard fork of [VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB). It has since diverged to focus on self-hosted operation, scanned-document workflows, MCP access, and conformance with the current draft of [Open Knowledge Format (OKF) v0.1](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md).

The architecture also draws on [Andrej Karpathy’s “LLM Wiki” concept](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): transforming source material into a persistent, interlinked knowledge base rather than reconstructing that knowledge for every query. okforge stores the resulting knowledge as portable Markdown with YAML frontmatter.

Special thanks to [SRI Consultants](https://sriconsultants.net/) for sponsoring the project’s development, contributing architectural ideas, and serving as its first production user.

**Licensing:** Apache-2.0 for the okforge engine; MIT for okforge-vision-ocr and okforge-webui.

