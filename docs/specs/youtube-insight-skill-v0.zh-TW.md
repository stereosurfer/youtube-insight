# YouTube Insight Extractor Skill V0 規格

日期：2026-05-22  
狀態：V0 草案，基於 LiSA 影片驗收題與 WebMCP 影片試跑修正  
目標：建立一個可換 agent、可換供應商、可持續重跑的單支 YouTube 內容提煉 Skill。

## 產品邊界

V0 只處理：

- 單支 YouTube URL。
- 或已存在的本地影片檔。
- 以知識型影片、投影片影片、產品 demo、技術解說為主要對象。
- 輸出個人知識庫用 Markdown 與 evidence artifacts。
- 可選擇掃描 YouTube 用戶回饋，用來找質疑、洞見、補充線索與反證。

V0 不處理：

- 多影片搜尋。
- 長期向量庫。
- 社群留言分析。
- Facebook / X / 網頁互動頁。
- PDF / 長文文件處理。
- 完整 VideoRAG。

## 平台可攜性

這套方法的核心分析流程其實不綁 YouTube。只要能取得以下任一種輸入，就可以套用：

- 本地影片檔。
- 可用工具取得的影片工作副本。
- 平台提供的字幕/章節/metadata。
- 使用者手動提供的影片、字幕或留言。
- 可由 browser/computer-use 看到的播放畫面。

真正會因平台不同而變的是 acquisition layer：

| 能力 | 平台相依性 | 說明 |
| --- | --- | --- |
| 影片畫面抽幀 | 中 | 只要能取得本地檔或可播放畫面，就能做；DRM/登入/短影音限制會增加難度。 |
| 字幕/章節 | 高 | YouTube 常有自動字幕；其他平台可能沒有、格式不同、或需要登入。 |
| 用戶回饋 | 高 | YouTube comments、Bilibili 彈幕/評論、Vimeo comments、論壇回覆都不同；V0 要抽象成 audience feedback artifact。 |
| metadata | 中 | 標題、作者、日期通常可取，但欄位與可信度不同。 |
| 外部查證 | 低 | 與平台無關，重點是主張本身。 |

因此 V0 命名仍以 YouTube 為主，是因為 YouTube 的 acquisition 最容易先做通。但 skill 內部要分層：

```text
platform adapter -> evidence inventory -> analysis pipeline -> knowledge artifact
```

未來只要新增平台 adapter，就能共用後面的 evidence inventory、observation、verification、insight synthesis。

本地 `yt-dlp --list-extractors` 顯示支援大量平台，包括 YouTube、Vimeo、Bilibili、TikTok、Twitch、Dailymotion、Facebook、Instagram 等，但這不代表每個平台都穩定可用；其中也有標示 currently broken 的 extractor。V1 不應承諾「大部分平台都能自動處理」，只能承諾「核心方法可攜，平台取得層需逐一驗證」。

## 重新調整後的流程順序

我建議把流程改成「低成本證據盤點先於模型精讀」：

```text
1. Intake
2. Cheap evidence inventory
3. Source type routing
4. Evidence extraction
5. Speech/visual alignment
6. Audience feedback scan
7. Observation journal
8. Claim and gap extraction
9. External verification
10. Insight synthesis
11. Delivery audit
```

### 為什麼調整

剛剛試跑時，我們先拿到字幕，再看 contact sheet。這對人類操作沒問題，但產品化時如果讓模型先讀字幕並開始總結，會容易被語音敘事牽著走。

但知識型影片本來就以語音說明與投影片文字為核心，所以也不該硬改成「視覺先行」。更好的順序是：

1. 先做便宜的 evidence inventory：metadata、字幕/章節、5 秒 frame/contact sheet。
2. 先知道它是 slide-driven、demo-driven、talking-head、hybrid，或 visual-heavy。
3. 再決定哪些段落需要 OCR、密集抽幀、視覺模型精讀、外部查證。

這樣才能避免兩個錯誤：一邊是只看字幕，漏掉畫面；另一邊是硬做視覺索引，忽略知識型影片真正依靠的說明文字。

## 階段設計

### 1. Intake

輸入：

- `youtube_url` 或 `local_video_path`
- optional: `user_focus`
- optional: `output_language`
- optional: `verification_level`

輸出：

- video id
- title
- duration
- source type
- working copy path
- transcript availability
- run id

規則：

- 不在最終輸出宣稱「已看完整影片」，除非 visual map 與 evidence coverage 都存在。
- working copy 可存在暫存區，但 final artifacts 必須保存必要 evidence。

### 2. Cheap Evidence Inventory

預設：

- 讀取影片 metadata。
- 取得字幕/章節；若沒有字幕，再視需求用 ASR。
- 每 5 秒抽 1 張 frame。
- 每 40 張 frame 做一張 5x8 contact sheet。
- 每格至少 320x180。

任務：

- 判斷影片類型。
- 找出投影片切換、圖表、表格、公式、demo UI、關鍵文字。
- 標記需要密集回看的時間段。

輸出：

- contact sheets
- transcript/chapters artifact
- audience feedback artifact, when enabled
- `visual_map.json`
- `evidence_inventory.json`

最低欄位：

```json
{
  "sampling_interval_s": 5,
  "video_type": "slide_driven",
  "candidate_segments": [
    {
      "start_s": 50,
      "end_s": 70,
      "reason": "new slide with API names",
      "needs_dense_review": false
    }
  ]
}
```

### 3. Source Type Routing

根據 evidence inventory 決定後續策略：

| 類型 | 策略 |
| --- | --- |
| slide-driven | 先用 5 秒抽樣或 scene/visual-diff 偵測建立初始切點，再 refined 成 slide map，每張 slide 保存高解析 keyframe |
| demo-driven | 對 UI 操作段落做 1s 或 0.5s 回看 |
| chart-heavy | 保存原始 frame + chart crop，必要時 OCR |
| talking-head | 主要用 transcript，但仍保存視覺變化/白板/螢幕文字 |
| mixed | 分段套用上述策略 |

### 4. Evidence Extraction

保存：

- contact sheets
- slide keyframes
- chart/table/formula crops
- demo sequence frames

規則：

- 原始 frame 是證據。
- crop 是閱讀輔助。
- 重建圖表是解讀，不可取代原圖。
- final insight 引用的視覺主張必須能回到 keyframe 或 crop。

### 5. Speech / Transcript Alignment

字幕在 evidence inventory 階段就可以取得，但只有在畫面結構也建立後，才進入模型精讀與合成。

任務：

- 對齊每個 visual segment 的 spoken claim。
- 標記講者主張與畫面證據是否一致。
- 找出只有語音、沒有畫面支持的主張。

欄位：

```json
{
  "segment_id": "seg_001",
  "spoken_claims": [],
  "visual_evidence_ids": [],
  "claim_support": "supported | unsupported | visual_missing | unclear"
}
```

### 6. Audience Feedback Scan

用戶回饋是獨立證據層，不屬於影片本體。它不能拿來證明「影片裡有說/有出現」，但可以用來找：

- 對影片主張的質疑。
- 觀眾提供的補充資料或來源。
- 實務經驗與反例。
- 更精準的術語、背景脈絡或 missing angle。
- 高讚數但可能錯誤的流行解讀。

輸入：

- YouTube comments / replies，若取得得到。
- 使用者手動貼上的留言、社群回覆、討論串。

輸出：

- `audience_feedback.json`

最低欄位：

```json
{
  "id": "fb_001",
  "source": "youtube_comment",
  "author_display": "redacted_or_public_handle",
  "engagement": {
    "likes": 120,
    "reply_count": 4
  },
  "feedback_type": "question | critique | correction | source_link | practitioner_experience | insight | noise",
  "text": "This is not actually shipped yet; it is an origin trial.",
  "related_video_claim_ids": ["claim_001"],
  "action": "verify | preserve_as_context | ignore",
  "privacy_note": "public_comment"
}
```

規則：

- 不把留言當事實來源，除非留言提供可查證來源或可驗證經驗。
- 高讚數只代表值得看，不代表正確。
- 有來源連結、具體反例、實務經驗、專業背景的留言優先保留。
- 情緒性支持/反對、梗、空泛稱讚預設降權。
- 若留言指出影片錯誤，必須進入 external verification。
- 最終筆記要明確標成「觀眾回饋線索」，不可混成影片主張。

### 7. Observation Journal

這是核心中間產物。它不是摘要。

每筆 observation 至少包含：

```json
{
  "id": "obs_001",
  "start_s": 55,
  "end_s": 65,
  "phase": "Tool Registration API",
  "visual_observation": "Slide shows Tool Registration API and mentions navigator.modelContext.registerTool().",
  "visible_text": ["Tool Registration API", "getTools()", "checkout", "search_products"],
  "spoken_claim": "Sites can register named tools with typed schemas.",
  "evidence_type": ["visual", "speech"],
  "source_artifacts": ["keyframes/frame_00-55.jpg", "transcript_compact.txt"],
  "inference": null,
  "confidence": "high",
  "needs_recheck": false
}
```

### 8. Claim And Gap Extraction

從 observation journal 抽出：

- 核心主張。
- 數字主張。
- 標準/版本/發布狀態。
- 產品/公司/人物/政策等易變事實。
- 明顯推論。
- 缺漏面向。
- 觀眾回饋指出的疑點與補充線索。

重要規則：

- 影片標題或講者語氣不等於事實。
- 看到「ships」「released」「first」「already」「all」這類字眼，預設進入查證。
- 數字和標準狀態必查。
- 觀眾回饋若提出具體反證或來源，也必查。

### 9. External Verification

查證不是把流程拖回舊 RAG，而是針對已抽出的高風險主張做最小必要查證。

優先來源：

1. 官方文件。
2. 標準草案 / RFC / W3C / API docs。
3. 公司官方公告。
4. 可信研究報告。

輸出：

```json
{
  "claim": "Chrome ships WebMCP",
  "source_in_video": "title and opening slide",
  "verification_result": "needs_rewording",
  "better_wording": "Chrome introduced/previewed WebMCP; origin trial starts in Chrome 149.",
  "sources": []
}
```

### 10. Insight Synthesis

輸入：

- observation journal
- claim/gap extraction
- verification results
- audience feedback findings
- user thinking style spec

輸出：

- 中文 Markdown。
- evidence index。
- confirmed / caution / unknown。
- personal knowledge note。

寫作原則：

- 不要只是摘要影片。
- 先講底層邏輯。
- 明確區分影片主張、畫面證據、外部查證、我的推論。
- 保留疑點。
- 區分影片內容、外部查證與觀眾回饋。

### 11. Delivery Audit

交付前檢查：

- 是否有 contact sheet？
- final insight 引用的視覺主張是否都有 keyframe/crop？
- 是否有 transcript artifact？
- 若啟用 audience feedback，是否明確分層並避免把留言當影片證據？
- 是否列出查證來源？
- 是否標記「未查證 / 推論 / 需要保留疑點」？
- 是否避免被影片標題或原作者 framing 牽走？

## V0 輸出結構

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

## 驗收清單

- [ ] 能處理 5 分鐘以內 slide-driven YouTube 影片。
- [ ] 能建立 metadata + transcript/chapters + contact sheets 的 evidence inventory。
- [ ] 若啟用留言掃描，能把觀眾回饋分成 question / critique / correction / source_link / practitioner_experience / insight / noise。
- [ ] 能把有價值的觀眾質疑與線索送進查證層。
- [ ] 最終 Markdown 能區分影片主張、視覺/語音證據、觀眾回饋、外部查證與我的推論。
- [ ] 能產生 contact sheets。
- [ ] 能產生每張重要投影片的 keyframe。
- [ ] 能把 transcript 視為 speech evidence，而不是唯一來源。
- [ ] 能輸出時間軸 observation table。
- [ ] 能標記數字/版本/標準狀態等高風險 claim。
- [ ] 能對至少 3 個高風險 claim 做外部查證。
- [ ] 能修正影片過度包裝的說法，例如 ships vs preview/origin trial。
- [ ] final Markdown 能連回 evidence artifacts。
- [ ] 報告中明確列出 confirmed / needs caution。

## 打包與可攜性

Codex 內建的 Skill Creator 比較像「給 Codex 自己用的技能包規格」，不是完整產品打包方案。V0 應拆成兩層：

1. **Codex Skill layer**：`skills/youtube-insight-extractor/SKILL.md`，描述代理人該如何執行流程。
2. **Portable product layer**：之後加入 `scripts/`、`schemas/`、`templates/`，讓 Claude Code、OpenCode、一般 CLI 或 web app 也能呼叫同一套流程。

建議未來打包結構：

```text
youtube-insight-extractor/
  SKILL.md
  agents/openai.yaml
  scripts/
    extract_frames.py
    build_contact_sheets.py
    extract_transcript.py
    build_evidence_inventory.py
    write_artifact.py
  schemas/
    evidence_inventory.schema.json
    audience_feedback.schema.json
    observation.schema.json
    verification.schema.json
  templates/
    insight.zh-TW.md
```

### 下一版可加

- slide-change detector，但作為 5 秒抽樣後的 refinement 或本機 visual-diff 初始切點，不是憑空取代抽樣。
- OCR-assisted slide text extraction。
- chart crop detection。
- local VLM adapter。
- Gemini native video adapter。
