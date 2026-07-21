# okforge

**Turn your scanned documents into a private, verifiable knowledge base—all on your own hardware.**

Open Knowledge Forge (okforge) is a local-first system designed to create a structured Knowledge Base (KB) from scanned documents and make that data accessible to a locally hosted Large Language Model (LLM) via the Model Context Protocol (MCP).

Most “chat with your documents” tools use a technique called RAG, or Retrieval-Augmented Generation. Think of it like giving an AI a few pages selected from a book rather than the full chapter. Those pages may contain the most relevant passages, but they can still omit important context, relationships, and supporting details—making the AI more likely to produce incomplete or unsupported answers.


**okforge** takes a different approach: it organizes your sources into a structured, interlinked digital wiki *before* you ever ask a question. It generates document summaries, maps key concepts, and extracts images—transforming raw scans into a curated library with page-level citations that let you trace generated information back to the original sources.

**Why this matters:**
*   **Traceable Answers:** Page-level citations let you inspect the original source and verify the AI’s interpretation for yourself.
*   **Privacy & Performance:** Because your data is organized in advance, capable smaller models can work with it efficiently on your own hardware. When okforge is configured with local inference, your documents and generated knowledge remain on systems you control.
*   **True Ownership:** Your knowledge is saved as plain Markdown files (compatible with apps like Obsidian). You aren't locked into a proprietary system—you own your data.

The wiki follows the current draft of the Open Knowledge Format [Open Knowledge Format (OKF)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md), keeping your structured knowledge portable and readable as standard Markdown. Through okforge’s MCP server, compatible AI clients can search and retrieve that knowledge.

## The Pipeline

<div align="center">
  <img src="../assets/pipeline.svg" width="1000" alt="okforge pipeline diagram">
</div>

### How It Works
1. **Extraction:** okforge uses vision-language models (VLMs) to transcribe scanned text, identify visual content, and preserve the relationship between each output and its original page. Images, diagrams, and photographs can be cropped and saved alongside the extracted text. Documents can also be translated during this stage.
2. **Structuring—the “Forge”:** Rather than producing a single long text file, okforge synthesizes the extracted material into an interlinked wiki. It identifies key concepts, creates summaries, and adds page-level citations that connect generated content to the source documents.
3. **Interaction:** Once the knowledge base is built, you can browse it as a website, search it from the command line, or connect it to an AI client through MCP. The model can locate relevant knowledge pages, read their curated content, and trace supporting information back to the original source pages.

## See a finished knowledge base

[**The Dade County Building Code of 1935**](https://okforge.github.io/dade-code-1935/) is a public knowledge base produced with **okforge** by [SRI Consultants](https://sriconsultants.net/), a Southeast Florida engineering firm that sponsored the system’s development.

The site demonstrates the output of the Forge stage: scanned documents transformed into an interlinked wiki with concept pages, full-text search, a graph view, extracted source material, and page-level citations.

For work involving historic buildings, engineers may need to consult the code under which a structure was originally permitted. SRI uses this knowledge base to research the materials and construction methods documented for historic Southeast Florida structures. Because the source document is publicly available, it also provides an accessible example of the same local-first workflow SRI uses for private processes and project information.


## The okforge Ecosystem

The system consists of two core Python packages and an optional reference Web interface. Each component can be used independently or as part of the complete document-to-knowledge pipeline.

### Core Tools

* [**okforge-vision-ocr**](https://github.com/okforge/okforge-vision-ocr) — **Document extraction**

  Transcribes scanned pages into Markdown using vision-language models, extracts images and diagrams, and records the source-page mappings needed for page-level citations.

  `pip install okforge-vision-ocr`

* [**okforge**](https://github.com/okforge/okforge) — **Knowledge-base generation and access**

  Compiles extracted or existing Markdown into a structured, interlinked wiki. It also provides command-line tools and an MCP server for searching, reading, and querying the resulting knowledge base.

  `pip install okforge`

### Reference Implementation

* [**okforge-webui**](https://github.com/okforge/okforge-webui) — **Local Web interface**

  Demonstrates how the core packages can be combined into a browser-based workflow with a PDF inbox, processing queue, knowledge-base browser, and publishing tools.

> **Note:** The Web UI is intended for local, single-user deployments and does not currently provide authentication or multi-user access controls.

    
## Knowledge in the Data, Not the Model Weights

Fine-tuning can adapt a model’s behavior, but it is not an ideal way to maintain factual knowledge that changes over time. **okforge** instead keeps domain information in structured, source-cited pages that the model reads when answering a question.

This approach allows a knowledge base to be updated without retraining a model and makes it possible to use the same content with different compatible models. Modern small and mid-sized models can work effectively with curated context, making local deployment practical for many document-analysis tasks.

Because the supporting pages retain citations to the original documents, users can trace an answer’s sources and evaluate the model’s interpretation for themselves.

## Choose the Components You Need

Each component can be used independently or as part of the complete pipeline:

* **Convert scanned PDFs to Markdown:** Use [`okforge-vision-ocr`](https://github.com/okforge/okforge-vision-ocr) to transcribe pages, extract visual content, and preserve source-page mappings. Its Markdown output can also be used with other document-processing or RAG systems.

* **Turn existing Markdown into a knowledge base:** Use [`okforge`](https://github.com/okforge/okforge) to organize source material into an interlinked, page-cited wiki. No OCR stage or Web interface is required.

* **Run the complete browser-based workflow:** Deploy [`okforge-webui`](https://github.com/okforge/okforge-webui) for an integrated PDF inbox, processing queue, knowledge-base browser, and publishing workflow.

## Bring your own chat client

okforge is not a chat app, and it ships without a vector database by design. Instead, it attaches to your preferred client (Open WebUI, llama.cpp, Page Assist, Hermes Agent, or any MCP-compatible host) as an **MCP server**.

This allows for a powerful **hybrid approach**: you can use your client's built-in Vector RAG for broad, semantic discovery while using okforge for precision and verification. By avoiding the embedding step for its own core logic, okforge uses a "locate-then-read" approach. The MCP tools provide full-text search over the wiki (`grep_wiki` / `search`), allowing the model to find and read curated, page-cited documents directly. 

Furthermore, a Knowledge Base in this system is a *subject collection*, not a wrapper around a single file. Many different documents can be combined into one KB; as new material lands, concept and entity pages accrete sources over time. The web UI's MCP server allows the model to search across every KB it hosts simultaneously.

## Local-first, for real

For many users, keeping data local is not a preference—it is a requirement. Ethical, legal, and proprietary constraints mean that sensitive business processes simply cannot be sent to a third-party cloud API. Everything in this suite is tuned for the case where the LLM resides on **your** hardware.

### Hardware Reality
The shift toward strong open multimodal models (such as the Qwen3.6-27B class) finally makes a fully local scan-to-wiki pipeline practical—provided the GPU has sufficient VRAM for both the model and a generous context window. 

This suite is developed and run daily on RTX 3090, RTX 5090, and RTX 6000 Pro Blackwell hosts; a single 24 GB card is a proven, real-world baseline, not a theoretical hope. To ensure stability on consumer hardware:
*   **Serial Ingest:** The job queue is strictly serial to avoid choking single-slot llama.cpp hosts.
*   **Zero Dependencies:** There is no telemetry, no account system, and no SaaS dependency in the path.
*   **Scaling:** More VRAM allows the host to serve multiple slots, and because queries are read-only, they parallelize across these slots so several clients can work the same KBs at once.

### Architectural Flexibility
okforge separates the management of knowledge from the compute required to process it. While a GPU is essential for the LLM to operate, it does not have to be local to the okforge installation. This flexibility allows you to deploy a modest server as a central knowledge hub for your entire LAN, offloading the inference to any OpenAI-compatible endpoint—whether that is a dedicated local GPU server or a hosted provider like OpenRouter.

## Start here

Ready to build your own verifiable knowledge base? 

*   **For developers:** Read the engine's [GETTING_STARTED](https://github.com/okforge/okforge/blob/main/GETTING_STARTED.md) guide.
*   **For a turnkey experience:** Clone [okforge-webui](https://github.com/okforge/okforge-webui) to launch the full point-and-click pipeline.
*   **Prefer to be walked through it?** Hand [this install prompt](https://github.com/okforge/okforge-webui/blob/main/docs/INSTALL_PROMPT.md) to a coding agent and it will set the suite up with you interactively.

---

### Project Heritage & Credits

The **okforge** engine began as a hard fork of [VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB). It diverges deliberately to prioritize self-hosted operation, scanned-document workflows, and strict OKF conformance over SaaS integration. For those who prefer a managed service and for whom local hosting is not a primary requirement, VectifyAI’s offering is an excellent choice.

The system follows Google's **Open Knowledge Format (OKF)**—an open specification that standardizes [Andrej Karpathy's "LLM Wiki" concept](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), ensuring that structured knowledge remains portable and readable by any modern AI client.

Special thanks to [SRI Consultants](https://sriconsultants.net/) for sponsoring the development of this project, contributing critical architectural ideas, and serving as its first real-world user.

**Licensing:** Apache-2.0 (engine) / MIT (vision-ocr, webui).
