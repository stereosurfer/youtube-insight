---
name: youtube-insight-extractor
description: Use when investigating a single YouTube URL or local video to produce an accountable Traditional Chinese personal knowledge note. The skill lets the model observe with modern video/vision/long-context/browser capabilities, requires an observation journal and reflection trace, then structures evidence and verifies high-risk claims before delivery.
---

# YouTube Insight Extractor

Use this skill for a single YouTube URL or local video when the user wants knowledge extraction, not a generic summary.

## Non-negotiables

- Freedom to observe. Mandatory trace. Accountable delivery.
- Do not force old ETL/RAG steps as the reasoning path: no DOM-first, OCR-first, chunk-first, Markdown-first, or fixed-table-first requirement.
- Use any available modern capability that fits the source: native video understanding, vision, audio, long context, screenshots, browser/computer-use, replay, zoom, interaction, search.
- Never claim the video was analyzed if only transcript was inspected. Label it `transcript-only`.
- Never describe an image, chart, slide, or demo UI as analyzed unless the trace shows visual observation.
- Never turn inference into fact.
- Treat audience feedback as leads, not proof.
- Keep product/design paths separate from user runtime paths. Runtime outputs go under user-configured roots only. Do not hard-code developer/designer paths.

## Required Inputs

- `youtube_url` or `local_video_path`
- `user_output_root`
- `user_knowledge_base_root`

Optional:

- `user_focus`
- `output_language`
- `verification_level`
- `audience_feedback_enabled`

If output roots are missing, ask for them or read explicit user configuration. Do not guess.

## Workflow

1. **Source Intake**
   - Identify source, title, duration, available modalities, acquisition limits, and output paths.
   - Do not start summarizing from transcript or title.

2. **Investigation Plan**
   - Decide how to inspect this source: full native video pass, contact sheets, replay, dense frame review, screenshot/crop, transcript alignment, comments, external search.
   - Name likely risks: title framing, release-status claims, numeric claims, chart ambiguity, prompt injection, missing visual layer.
   - Write `{user_output_root}/youtube-insight-runs/{source_id}/investigation_plan.md`.

3. **Open Observation**
   - Observe the source using the planned capabilities.
   - Save helpful artifacts: contact sheets, keyframes, crops, transcript, comment leads.
   - Rewatch, zoom, interact, or compare states when needed.
   - Record missing evidence instead of filling gaps.

4. **Observation Journal**
   - Write `{user_output_root}/youtube-insight-runs/{source_id}/observation_journal.md`.
   - This is the core artifact. It is not a summary.
   - Each entry must say: time/location, what was seen/heard, why it mattered, whether it was rechecked, uncertainty, and whether it is observation, source claim, audience lead, external fact, or inference.

5. **Reflection / Red Team**
   - Write `{user_output_root}/youtube-insight-runs/{source_id}/reflection.md`.
   - Ask: Was I led by framing? Did I infer from transcript without visual support? Did I miss charts/demo UI/version status? What changed after rewatching? What still needs checking?

6. **Evidence Structuring**
   - Derive structure from the journal, not the other way around.
   - Produce only useful derived files: `evidence_index.json`, `claim_map.json`, `verification_questions.json`, `audience_feedback.json`.
   - High-risk claims include numeric, release/version/status, standard, company/product/policy, and words like `ships`, `released`, `first`, `already`, `all`, `standard`.

7. **External Verification**
   - Verify high-risk claims with primary sources where possible.
   - Record confirmed, contradicted, needs rewording, not verified.
   - Do not use search to replace observation; use it to check claims.

8. **Final Synthesis**
   - Write the final Traditional Chinese note under `{user_knowledge_base_root}`.
   - Include bottom-line logic, observation-based findings, verified facts, cautions, corrected framing, personal knowledge insight, and evidence links.
   - Separate observation, source claim, audience lead, external verification, and inference.

9. **Delivery Honesty Check**
   - Every important final claim must trace back to the journal or verification.
   - If visual evidence is missing, say so.
   - If transcript-only, do not present as full video analysis.
   - If a statement is uncertain, keep it uncertain.

## Default Artifact Layout

```text
{user_output_root}/youtube-insight-runs/{source_id}/
  investigation_plan.md
  observation_journal.md
  reflection.md
  evidence/
    contact_sheets/
    keyframes/
    crops/
    transcript_compact.txt
    audience_feedback.json
  data/
    evidence_index.json
    claim_map.json
    verification_questions.json
    verification.json

{user_knowledge_base_root}/
  youtube-insight-{source_id}-{date}.zh-TW.md
```

## Acceptance Checks

- Output paths are user-configured; no developer/designer home path is hard-coded.
- Runtime artifacts are not written into the product repo unless explicitly requested as fixtures.
- Investigation plan exists and states the observation strategy.
- Observation journal exists and records actual observation, rechecks, uncertainty, and evidence type.
- Reflection exists and challenges framing, transcript-only inference, missed visuals, and unsupported certainty.
- Derived JSON files come after the journal; they do not replace the journal.
- Audience feedback, if used, is separated from video evidence and classified as leads.
- Important visual claims trace to observed frames, crops, native-video observations, or explicit visual notes.
- High-risk claims are verified or explicitly marked unverified.
- Final note does not exceed the trace.

## Style

Write for the user's personal knowledge base:

- concise Traditional Chinese,
- direct distinction between fact, source claim, and inference,
- no marketing summary tone,
- preserve useful uncertainty.
