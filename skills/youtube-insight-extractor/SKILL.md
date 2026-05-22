---
name: youtube-insight-extractor
description: Use when extracting an evidence-backed personal knowledge note from a single YouTube URL or local video. The skill builds a cheap evidence inventory from metadata, transcript/chapters, visual contact sheets, and optional audience feedback before model-led reasoning, verifies high-risk claims, and writes a Traditional Chinese insight artifact with links to evidence.
---

# YouTube Insight Extractor

Use this skill for a single YouTube URL or local video when the user wants knowledge extraction, not a generic summary.

The analysis pipeline is platform-portable, but acquisition is platform-specific. For non-YouTube platforms, first verify that video frames, transcript/chapters, metadata, and optional audience feedback can be obtained or supplied by the user. If acquisition is incomplete, label the missing evidence layer.

## Core Rule

Build a cheap evidence inventory before model-led reasoning.

Never claim the video was analyzed if the run has no visual evidence artifacts. Transcript-only output must be labeled transcript-only and is not acceptable for final insight unless the user explicitly asks for a transcript summary.

## Workflow

1. **Intake**
   - Capture URL/path, title, duration, date, and run id.
   - Create a working copy or use the provided local file.
   - Store artifacts under `artifacts/youtube-insight-runs/{video_id}/`.

2. **Cheap Evidence Inventory**
   - Capture transcript/chapters when available, but do not summarize from transcript alone.
   - Sample one frame every 5 seconds.
   - Build 5x8 contact sheets, 40 frames per sheet, each tile at least 320x180.
   - If audience feedback is available or requested, collect comments/replies as a separate evidence layer.
   - Classify the video: `slide_driven`, `demo_driven`, `chart_heavy`, `talking_head`, or `mixed`.
   - Identify candidate segments for slides, charts, tables, formulas, demo UI, key text, or unclear areas.

3. **Route By Type**
   - `slide_driven`: use 5s sampling or local visual-diff/scene detection to build an initial slide map, then save one high-resolution keyframe per important slide.
   - `demo_driven`: dense review at 1s or 0.5s for operation segments.
   - `chart_heavy`: save original frame plus chart/table crops.
   - `talking_head`: rely more on speech evidence, but still preserve visual changes and on-screen text.
   - `mixed`: apply segment-specific routes.

4. **Align Speech**
   - Use transcript after the evidence inventory exists.
   - Align spoken claims to visual segments.
   - Mark whether a claim is supported by visual evidence, speech only, unsupported, or unclear.

5. **Audience Feedback Scan**
   - Treat audience feedback as leads, not facts.
   - Classify feedback as `question`, `critique`, `correction`, `source_link`, `practitioner_experience`, `insight`, or `noise`.
   - Preserve useful doubts, concrete counterexamples, source links, and practitioner context.
   - Send concrete corrections or source links into verification.
   - Never use comments as proof that something appears in the video.

6. **Observation Journal**
   - Produce timestamped observations, not a summary.
   - Each observation should include start/end, phase, visible text, visual observation, spoken claim, evidence artifacts, inference, confidence, and `needs_recheck`.

7. **Claim And Gap Extraction**
   - Extract core claims, number claims, version/release/standard claims, company/product claims, inferred claims, and missing dimensions.
   - Include audience feedback leads that raise concrete doubts or new angles.
   - Treat words like `ships`, `released`, `first`, `already`, `all`, `standard`, and numeric claims as high-risk.

8. **External Verification**
   - Verify high-risk claims using primary sources where possible.
   - Prefer official docs, standards drafts, company announcements, and credible reports.
   - Record confirmed, contradicted, needs rewording, and not verified.

9. **Insight Artifact**
   - Write Traditional Chinese Markdown under `docs/runs/`.
   - Include evidence links, timeline observation, audience feedback findings when available, confirmed/caution sections, insight, personal knowledge note, and trial/result notes.
   - Distinguish visual evidence, speech claims, audience feedback, external verification, and inference.

## Default Artifact Layout

```text
artifacts/youtube-insight-runs/{video_id}/
  contact_sheets/
  keyframes/
  crops/
  transcript_compact.txt
  audience_feedback.json
  evidence_inventory.json
  visual_map.json
  observations.json
  verification.json

docs/runs/
  youtube-insight-{video_id}-{date}.zh-TW.md
```

## Acceptance Checks

- Contact sheets exist.
- Transcript/chapters are present when available, or ASR absence is noted.
- Audience feedback, if used, is separated from video evidence and classified by value.
- Important visual claims in the final note link to keyframes or crops.
- Transcript is not the only evidence layer.
- At least the high-risk claims are verified or explicitly marked unverified.
- The final note corrects source framing when evidence requires it.
- Known gaps and uncertainty are visible in the artifact.

## Style

Write for the user's personal knowledge base:

- concise Traditional Chinese,
- direct distinction between fact, source claim, and inference,
- no marketing summary tone,
- preserve useful uncertainty.
