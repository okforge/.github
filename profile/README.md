# okforge

**Scanned books in; a citation-backed, LLM-synthesized wiki out — all on
your own hardware.**

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

## Local-first, for real

Everything is tuned for the case where the LLM is **your** machine, not
a cloud API: a strictly serial job queue that won't choke a single-slot
llama.cpp host, measured guidance for thinking-model footguns, and no
telemetry, accounts, or SaaS dependencies anywhere in the path. Any
OpenAI-compatible endpoint works — local llama.cpp/vLLM, or hosted
providers like OpenRouter through the same model string.

## Start here

New to the suite? Read the engine's
[GETTING_STARTED](https://github.com/okforge/okforge/blob/main/GETTING_STARTED.md),
or clone [okforge-webui](https://github.com/okforge/okforge-webui) for
the full point-and-click pipeline.

---

The engine began as a hard fork of
[VectifyAI/OpenKB](https://github.com/VectifyAI/OpenKB) and diverges
deliberately — self-hosted operation, OKF conformance, and scanned-book
workflows over SaaS integration. Apache-2.0 (engine) / MIT (webui).
