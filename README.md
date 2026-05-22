# YouTube Insight

**Observe, Structure, Investigate, Verify, Report.**

YouTube Insight turns knowledge videos into evidence-backed notes for a personal knowledge base.

The project is currently a V0 specification plus a Codex Skill draft. It focuses on one narrow workflow:

```text
YouTube URL or local video
-> cheap evidence inventory
-> visual/keyframe evidence package
-> transcript and audience-feedback alignment
-> claim and gap extraction
-> external verification
-> Traditional Chinese insight note
```

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

Build a cheap evidence inventory before model-led reasoning.

The transcript is useful, but it is not enough. Visual evidence, slide text, charts, demo screens, audience feedback, and external sources should stay in separate layers until synthesis.

## Repository Map

- [`skills/youtube-insight-extractor/SKILL.md`](skills/youtube-insight-extractor/SKILL.md): Codex Skill draft.
- [`docs/specs/youtube-insight-skill-v0.zh-TW.md`](docs/specs/youtube-insight-skill-v0.zh-TW.md): V0 product and workflow spec.

## Status

This is an early design and skill-packaging repository. The next step is to turn the workflow into portable scripts and schemas so it can run outside a single Codex session.
