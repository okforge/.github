# okforge

**Turn your scanned documents into a private, verifiable knowledge base—all on your own hardware.**

Open Knowledge Forge (okforge) is a local-first system designed to create a structured Knowledge Base (KB) from scanned documents and make that data accessible to a locally hosted Large Language Model (LLM) via the Model Context Protocol (MCP).

Most "chat with your documents" tools work by feeding an AI random fragments of text and hoping for the best. This often leads to hallucinations or answers that lack context. 

**okforge** takes a different approach: it organizes your sources into a structured, interlinked digital wiki *before* you ever ask a question. It generates document summaries, maps out key concepts, and extracts images—transforming raw scans into a curated library where every claim is backed by a real page citation `(p. N)`.

**Why this matters:**
*   **Trust through Verifiability:** You don't have to guess if the AI is right; you can see exactly which page of your original document the information came from.
*   **Privacy & Performance:** Because your data is pre-organized, you can use smaller, private AI models on your own computer without sacrificing quality. Your data never leaves your machine.
*   **True Ownership:** Your knowledge is saved as plain Markdown files (compatible with apps like Obsidian). You aren't locked into a proprietary system—you own your data forever.

The wiki follows the [Open Knowledge Format (OKF)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/SPEC.md), ensuring that your structured knowledge is portable, standardized, and readable by any modern text editor or MCP-compatible AI client.

## The Pipeline

<div align="center">
  <img src="../assets/pipeline.svg" width="1000" alt="okforge pipeline diagram">
</div>

### How it Works

1.  **Extraction:** okforge uses Vision-Language Models (VLM) to not only read the text (OCR) but also recognize and extract images, diagrams, and photos from your scans. If your documents are in another language, they can be translated during this stage.
2.  **Structuring (The "Forge"):** Instead of just saving a long text file, okforge "forges" the data into an interlinked wiki. It identifies key concepts, creates summaries, and ensures every piece of information is tagged with its original page number.
3.  **Interaction:** Once your knowledge base is built, you can use it however you like: browse it as a personal website, search it via command line, or connect it to a local AI model (via MCP) to chat with your data with high confidence.

## See a finished wiki

[**The Dade County Building Code of 1935**](https://okforge.github.io/dade-code-1935/)
— a real working example from engineering practice. When supporting
work on historic buildings, what matters is the code the structure was
originally permitted under — and those old editions exist only as
scans. Here the 1935 Miami-Dade building code (a public record) has
been OCR'd and compiled into a browsable wiki: cross-document concept
and entity pages, full-text search, a graph view, and `(p. N)` page
citations throughout.

## Repositories

| repo | what it does |
|---|---|
| [**okforge**](https://github.com/okforge/okforge) | The engine: compiles documents into an OKF-conformant, citation-backed wiki; `query`, `chat`, and an MCP server over any KB. `pip install okforge` |
| [**okforge-vision-ocr**](https://github.com/okforge/okforge-vision-ocr) | Pre-conversion: one vision-LLM call per page produces a Markdown transcription **and** photo/figure crops **and** the page map the engine reads for real citations. Table mode, translation workflow. Any OpenAI-compatible VLM. `pip install okforge-vision-ocr` |
| [**okforge-webui**](https://github.com/okforge/okforge-webui) | LAN web UI + job runner: drop a PDF in an inbox and drive it through probe → pilot → OCR → ingest → verify, with a serial job queue built for single-slot LLM hosts, an MCP endpoint, and one-button [Quartz](https://quartz.jzhao.xyz/) site publishing. |

## Small models, big context — no fine-tuning

Getting your domain into a local model used to mean fine-tuning or a
LoRA: expensive, frozen at training time, tied to one model, and
impossible to cite. A compiled KB is the alternative — the knowledge
stays *out of the weights*, in well-structured, grounded, traceable
pages the model reads at inference. The large context windows of
current open-source models are what make this practical: whole curated
pages fit in context, so a 7–30B model answers from finished synthesis
instead of reconstructing it. Updating the KB means adding a document,
not retraining; switching models keeps the KB; and every answer traces
to a page.

## Take only what you need

The three repos are a pipeline, not a bundle — each layer is useful on
its own:

- **Just want PDFs as clean Markdown?** `pip install okforge-vision-ocr`
  converts scanned pages to Markdown with extracted photos and a page
  map — use it in front of any downstream tool, no engine required.
- **Already have Markdown documents?** `pip install okforge` compiles
  them straight into a citation-backed wiki — no OCR step, no web UI.
- **Want the full point-and-click pipeline?** Add okforge-webui on top
  for the inbox, job queue, wiki browser, and one-button publishing.

## Bring your own chat client

okforge is not a chat app and ships no vector database — by design. It
attaches to the client you already run (Open WebUI, llama.cpp's WebUI,
Page Assist, Claude, or any MCP client) as an MCP server, alongside
that client's own built-in RAG. The MCP tools include full-text search
over the wiki (`grep_wiki` / `search`), so models locate-then-read
curated, page-cited pages — no embedding step anywhere. And a KB is a
subject collection, not a wrapper around a single file: many documents
combine into one KB, concept and entity pages accrete sources as new
material lands, and the web UI's MCP server searches across every KB
it hosts.

## Local-first, for real

For many of the people this suite was built for, keeping data local is
not a preference — it's a requirement. Ethical, legal, privacy, and
proprietary constraints mean documents and business processes
sometimes simply cannot go to a third-party service. So everything here is tuned
for the case where the LLM is **your** machine, not a cloud API. Strong open multimodal models in the Qwen3.6-27B class
finally make a fully local scan-to-wiki pipeline practical — provided
the GPU has room for the model *and* generous context. The suite is
developed and run daily on RTX 3090, RTX 5090, and RTX 6000 Pro
Blackwell hosts, so a single 24 GB card is a lived configuration, not
a hope. A strictly serial job queue won't choke a single-slot
llama.cpp host, thinking-model footguns come with measured guidance,
and there is no telemetry, account, or SaaS dependency anywhere in the
path. It scales up as well as down: more VRAM means the model host can
serve multiple slots, and queries — read-only by design — parallelize
across them today, so several clients can work the same KBs at once.
Ingest is deliberately serial for now; parallel and multi-card ingest
support is in active development.

And none of the pieces need to share a machine. okforge itself needs
no GPU — it runs on any box that can reach an LLM endpoint over the
network, and clients reach okforge the same way. A modest server
between your desktop and the GPU machine can host the KBs, the web UI,
and the MCP server for the whole LAN. Any OpenAI-compatible endpoint works — local llama.cpp/vLLM, or
hosted providers like OpenRouter through the same model string.

## Start here

New to the suite? Read the engine's
[GETTING_STARTED](https://github.com/okforge/okforge/blob/main/GETTING_STARTED.md),
or clone [okforge-webui](https://github.com/okforge/okforge-webui) for
the full point-and-click pipeline.

---

The engine began as a hard fork of
[VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB) and diverges
deliberately — self-hosted operation, OKF conformance, and
scanned-document workflows over SaaS integration. Apache-2.0 (engine) / MIT (webui).
