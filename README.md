# YouTube Insight

**Observe, Structure, Investigate, Verify, Report.**

YouTube Insight turns knowledge videos into accountable investigation notes for a personal knowledge base.

The project is currently a V0 specification plus a Codex Skill draft. It focuses on one narrow workflow:

```text
YouTube URL or local video
-> investigation plan
-> open observation
-> observation journal
-> reflection and recheck
-> evidence structuring
-> final accountable note
```

## Path Boundary

The product repository is only for product assets:

- specs,
- skills,
- schemas,
- templates,
- source code.

Runtime output belongs to the user's configured folders, not this repository and not any developer/designer home path.

Generated notes, contact sheets, keyframes, transcripts, verification logs, downloads, and working copies must be written under a user-provided output root or knowledge-base root. Examples in this repo use placeholders such as `{user_output_root}` and `{user_knowledge_base_root}` on purpose.

## Current Scope

V0 supports the product design for:

- single YouTube videos,
- local video files as fallback input,
- knowledge videos, slide videos, technical explainers, and product demos,
- optional audience feedback scan as a separate evidence layer,
- Traditional Chinese Markdown output for personal knowledge bases.

V0 does not aim to support:

- multi-video search,
- long-term vector indexing,
- full VideoRAG,
- generic social media crawling,
- PDF or long-document ingestion.

## Key Principle

Freedom to observe. Mandatory trace. Accountable delivery.

The model is allowed to use modern capabilities: video understanding, long context, vision, audio, screenshots, browser/computer-use, interaction, zooming, replaying, and external search.

The product does not force old ETL steps such as DOM extraction, OCR-first processing, chunk-first RAG, or fixed claim tables as the main reasoning path.

What the product enforces is honesty:

- say what was observed,
- say how it was known,
- say what was not observed,
- mark inference as inference,
- recheck weak or surprising claims,
- keep the final note faithful to the trace.

## Repository Map

- [`skills/youtube-insight-extractor/SKILL.md`](skills/youtube-insight-extractor/SKILL.md): Codex Skill draft.
- [`docs/specs/investigation-first-knowledge-extraction-v0.zh-TW.md`](docs/specs/investigation-first-knowledge-extraction-v0.zh-TW.md): V0 product and workflow spec.

## Status

This is an early design and skill-packaging repository. The next step is to turn the workflow into portable scripts and schemas so it can run outside a single Codex session.
