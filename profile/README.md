# okforge

**Scanned documents in; a citation-backed, LLM-synthesized wiki out —
all on your own hardware.**

Most "chat with your documents" tools hand an LLM a pile of raw chunks
at query time and hope. okforge instead **compiles** your sources ahead
of time into an interlinked Markdown wiki — per-document summaries,
cross-document concept and entity pages, extracted images — where every
claim carries a real `(p. N)` citation back to the original scan. Small
models running on your own llama.cpp/vLLM box answer from curated pages
instead of reconstructing from chunks, and what they say is checkable.

The wiki is plain Markdown with typed frontmatter, following the
[Open Knowledge Format (OKF)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md):
readable in Obsidian or any editor, queryable from a CLI, chat REPL, or
any MCP client, and publishable as a static site. Portable — not locked
to okforge itself.

## The pipeline

```
scanned PDF ──▶ VLM OCR + photo extraction ──▶ (optional translation)
           ──▶ ingest ──▶ interlinked wiki with page citations
           ──▶ browse · query · chat · MCP · publish static site
```

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
support is in active development. Any OpenAI-compatible endpoint works — local llama.cpp/vLLM, or
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
