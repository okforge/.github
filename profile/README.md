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

## See a Finished Knowledge Base

[**The Dade County Building Code of 1935**](https://okforge.github.io/dade-code-1935/) is a public knowledge base produced with **okforge** by [SRI Consultants](https://sriconsultants.net/), a Southeast Florida engineering firm that sponsored the system’s development.

The site demonstrates the output of the Forge stage: scanned documents transformed into an interlinked wiki with concept pages, full-text search, a graph view, extracted source material, and page-level citations.

For work involving historic buildings, engineers may need to consult the code under which a structure was originally permitted. SRI uses this knowledge base to research the materials and construction methods documented for historic Southeast Florida structures. Because the source document is publicly available, it also provides an accessible example of the same local-first workflow SRI uses for private processes and project information.


## The okforge Ecosystem

The system consists of two core tools and a reference interface to demonstrate how they work together.

### Core Tools
These are primary Python libraries used to process documents and interact with your knowledge base. Both can be run directly from the command line or imported as modules into your own applications and scripts.

*   [**okforge-vision-ocr**](https://github.com/okforge/okforge-vision-ocr) $\rightarrow$ **The Digitizer**  
    The first step in the pipeline. It uses Vision AI to transcribe scans into clean text, crop out images and diagrams, and create the precise page maps required for real page number citations. 
    `pip install okforge-vision-ocr`

*   [**okforge**](https://github.com/okforge/okforge) $\rightarrow$ **The Knowledge Engine**  
    The heart of the system. It compiles digitized text into a structured, interlinked wiki and provides the interfaces (`chat`, `query`, and MCP server) to let you talk to your data via local AI models.
    `pip install okforge`

### Reference Implementation
*   [**okforge-webui**](https://github.com/okforge/okforge-webui) $\rightarrow$ **Local Dashboard (Example)**  
    This is a reference implementation demonstrating how the core tools can be combined into a usable application. It is designed for local, single-user use, allowing you to drop PDFs into an inbox and drive them through the pipeline via a browser interface. 

    *Note: As this is a demo for local setups, it does not include multi-user support or authentication.*
    
## Data-Driven Intelligence, Not Model-Baked Knowledge

Adding domain knowledge via fine-tuning or LoRAs is expensive, static, and opaque. **okforge** avoids this by keeping knowledge *out of the weights* and in structured, traceable pages that the model reads at inference. 

This approach is made possible by a new generation of highly capable small models. These models are not just larger in context window, but fundamentally "smarter"—they can reason over complex, unfamiliar data they weren't trained on with remarkable precision. Consequently, a 7–30B model can answer based on pre-synthesized curated pages rather than relying on flawed internal memory. This ensures that updating the KB is as simple as adding a document, switching models is seamless, and every response remains strictly verifiable.

## Modular by Design

The three repositories are components of a pipeline, not a monolithic bundle—each layer provides standalone value:

*   **Just want PDFs as clean Markdown?** Use [`okforge-vision-ocr`](https://github.com/okforge/okforge-vision-ocr). It converts scanned pages to Markdown with extracted photos and page maps. Because it produces standardized Markdown, its output can be used directly in any other downstream tool or ingested into traditional RAG systems.
*   **Already have Markdown documents?** Use [`okforge`](https://github.com/okforge/okforge). It compiles existing text directly into a citation-backed wiki—no OCR step or web UI necessary.
*   **Want the full point-and-click pipeline?** Deploy [`okforge-webui`](https://github.com/okforge/okforge-webui) on top for an integrated inbox, job queue, wiki browser, and one-button publishing.

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
