# Investigation-first Knowledge Extraction Spec V0

日期：2026-05-22  
狀態：V0 草案  
產品：YouTube Insight  
目標：把單支 YouTube 或本地影片轉成可追溯、可換 agent、可換供應商的個人知識庫筆記。

## 一句話

自由觀察，強制留痕，交付前問責。

## 產品邊界

V0 支援：

- 單支 YouTube URL。
- 已存在的本地影片檔。
- 知識型影片、投影片影片、技術解說、產品 demo。
- 可選的 YouTube 用戶回饋掃描。
- 繁體中文 Markdown 筆記。

V0 不支援：

- 多影片搜尋。
- PDF / 長文文件處理。
- 泛社群爬取。
- 長期向量庫。
- VideoRAG。
- 自動處理所有影音平台。

## 設計端 / 用戶端路徑邊界

硬規則：產品 repo 只放產品本體，不放使用者執行後的資料。

產品 repo 可包含：

- README
- spec
- skill
- schema
- template
- source code

產品 repo 不應包含：

- 下載的影片工作副本
- contact sheets
- keyframes / crops
- transcript runtime output
- observation / verification runtime JSON
- 使用者的最終知識庫筆記
- 單次執行的 evidence package

執行輸出只能寫到使用者指定的位置：

- `{user_output_root}`：單次執行的 evidence package。
- `{user_knowledge_base_root}`：最終 Markdown 筆記。
- `{runtime_temp_root}`：可刪除暫存。

禁止在產品規格、Skill、範例中寫死任何開發者或設計者家目錄。若使用者沒有提供輸出位置，必須要求設定或讀取明確使用者設定，不得猜測。

## 核心原則

### 1. 驗證層不規定模型怎麼看

不要把流程寫死成：

```text
source -> parse -> schema -> summary
```

也不要預設：

- 一定要抽 DOM。
- 一定要 OCR。
- 一定要切 chunk。
- 一定要先轉 Markdown。
- 一定要填固定欄位後才准思考。

### 2. 驗證層只要求留下可回放痕跡

允許模型使用新能力：

- 看影片、音訊、畫面脈絡。
- 看整頁畫面或播放畫面。
- 操作互動元件。
- 滾動、展開、比較不同狀態。
- 用長上下文吸收大量材料。
- 自己決定哪裡值得放大、重看、查證。

但必須留下：

- 看了哪些畫面、時間段或狀態。
- 做過哪些互動或重看。
- 哪些地方有圖、圖表、影片、留言、引用。
- 哪些地方不確定。
- 哪些地方只是推測。
- 哪些地方可能受原始 framing 或 prompt injection 污染。

### 3. 最終交付要被問責

不准：

- 把沒看過的東西寫成看過。
- 把推測寫成事實。
- 只看字幕就說看完影片。
- 只讀圖片標題就說分析過圖片。
- 原文是摘要或宣傳文就順著寫。
- 省略重要不確定性。

## V0 工作流

```text
1. Source Intake
2. Investigation Plan
3. Open Observation
4. Observation Journal
5. Reflection / Red Team
6. Evidence Structuring
7. External Verification
8. Final Synthesis
9. Delivery Honesty Check
```

### 1. Source Intake

輸入：

- `youtube_url` 或 `local_video_path`
- `user_output_root`
- `user_knowledge_base_root`
- optional: `user_focus`
- optional: `output_language`
- optional: `verification_level`
- optional: `audience_feedback_enabled`

輸出：

- source id
- title
- duration
- available modalities
- acquisition limits
- output paths

規則：

- 不急著轉格式。
- 不因為有字幕就開始摘要。
- 若某個證據層拿不到，記錄缺口，不假裝看過。

### 2. Investigation Plan

模型先判斷這支素材應該怎麼看。

必答問題：

- 這像是 slide-driven、demo-driven、talking-head、chart-heavy、mixed，還是未知？
- 哪些證據層目前可用：畫面、音訊、字幕、章節、留言、外部來源？
- 哪些段落最可能含有關鍵文字、圖表、demo、規格、數字或高風險主張？
- 需要先粗看、直接精看、還是邊看邊回放？
- 哪些地方容易被標題、講者語氣、留言或頁面指令污染？

輸出：`investigation_plan.md`

### 3. Open Observation

這一步是自由探索，不是填表。

允許：

- 用原生影片理解能力直接看影片。
- 用抽幀/contact sheet 輔助定位。
- 放大或重看可疑片段。
- 對 demo 段落做密集回看。
- 對圖表、表格、投影片保存截圖或 crop。
- 掃描用戶回饋尋找質疑、洞見、反證或來源連結。

規則：

- contact sheet 是定位工具，不是唯一流程。
- transcript 是 speech evidence，不是影片本體的替代品。
- OCR、ASR、JSON 都是輔助，不是主導思路。
- 用戶回饋是線索，不是影片證據。

### 4. Observation Journal

核心中間產物：`observation_journal.md`

每筆觀察用自然語言記錄，至少包含：

- 時間段或畫面位置。
- 我看到 / 聽到什麼。
- 我為什麼注意到它。
- 它支持或挑戰了什麼初始判斷。
- 我是否重看、放大、展開或比較過。
- 哪裡仍不確定。
- 這是觀察、講者主張、觀眾線索、外部資料，還是我的推論。

範例：

```markdown
## obs-007 | 03:35-03:45

畫面是 IDE/demo，不是投影片。講者提到 custom agents 的設定位置。
我重看一次，因為畫面中的路徑文字停留時間短，而且可能影響可操作性。
可見文字包含 `~/.codex/agents`，但設定欄位沒有全部看清楚。

狀態：觀察 + speech claim
不確定：設定檔格式與欄位名稱需要官方文件查證
下一步：保存 keyframe，將「支援 custom agents」送進查證
```

### 5. Reflection / Red Team

回頭攻擊自己的觀察。

必問：

- 我是否被影片標題、開場、留言或作者 framing 帶著走？
- 我是否只看字幕就推論了畫面內容？
- 我是否把視覺上沒確認的內容寫成確認？
- 我是否漏看圖表、表格、demo UI、版本狀態或限制條件？
- 哪些初始判斷在觀察後改變？
- 哪些主張需要回去重看？

輸出：`reflection.md`

### 6. Evidence Structuring

只在 observation journal 之後做結構化。

可產生：

- `evidence_index.json`
- `claim_map.json`
- `verification_questions.json`
- `audience_feedback.json`

這些 JSON 是從觀察紀錄派生，不是用來綁死觀察過程。

最低結構：

```json
{
  "id": "claim-001",
  "claim": "The feature is shipped in Chrome.",
  "source": "video_title_or_spoken_claim",
  "supporting_observations": ["obs-002", "obs-004"],
  "evidence_type": ["speech", "visual_text"],
  "risk": "high",
  "why_verify": "release status claim"
}
```

### 7. External Verification

查證只針對高風險或有疑點的主張，不把整個系統拖回舊式 RAG。

必查：

- 數字。
- 版本。
- 發布狀態。
- 標準狀態。
- 公司 / 產品 / 政策現況。
- 觀眾提出具體反證或來源的點。
- 「ships」「released」「first」「already」「all」「standard」類字眼。

優先來源：

1. 官方文件。
2. 標準草案 / RFC / W3C / API docs。
3. 公司官方公告。
4. 可信研究報告。

輸出：`verification.md` 或 `verification.json`

### 8. Final Synthesis

輸出：繁體中文 Markdown 筆記。

必須包含：

- 一句話底層邏輯。
- 觀察摘要，不是影片摘要。
- 已確認事實。
- 需要保留的疑點。
- 被修正的原始 framing。
- 對個人知識庫有用的洞察。
- 來源與 evidence artifact 連結。

寫作規則：

- 明確區分：我看到的、講者主張、觀眾線索、外部查證、我的推論。
- 不用行銷摘要語氣。
- 不為了完整感補不存在的證據。

### 9. Delivery Honesty Check

交付前回答：

- 最終文章每個關鍵主張是否能回到 observation journal？
- 有沒有把未確認內容寫成確認？
- 有沒有說「看完影片」但實際只有字幕或少量抽幀？
- 圖片、圖表、demo UI 的描述是否有對應畫面證據？
- audience feedback 是否被標成線索，而不是事實？
- 是否保留未看、看不清、沒查到、需重看的地方？

若不通過，回到 Open Observation 或 Reflection。

## 預設輸出結構

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

## 驗收清單

- [ ] 使用者輸出路徑由 `user_output_root` / `user_knowledge_base_root` 決定，沒有硬編碼開發者路徑。
- [ ] 產品 repo 不保存 runtime artifacts，除非使用者明確要求 sample fixture。
- [ ] 產生 `investigation_plan.md`，且其中有明確觀察策略。
- [ ] 產生 `observation_journal.md`，且不是摘要。
- [ ] journal 記錄看過的時間段、重看/放大/互動、疑點與不確定。
- [ ] 若只取得 transcript，最終輸出必須標成 transcript-only，且不得稱為完整影片分析。
- [ ] 圖表、投影片、demo UI 的重要描述能回到 keyframe、crop 或觀察紀錄。
- [ ] 用戶回饋若使用，必須獨立分層，不能當影片證據。
- [ ] 產生 reflection，並檢查 framing 污染、漏看畫面、推測冒充事實。
- [ ] JSON 結構化產物由 journal 派生，不先用固定 schema 綁住觀察。
- [ ] 高風險主張進入外部查證或明確標示未查證。
- [ ] 最終 Markdown 明確區分觀察、講者主張、觀眾線索、外部查證、推論。
- [ ] final note 忠於 trace；沒有 trace 支撐的內容不可寫成事實。

## 可攜性

V0 分兩層：

1. **Codex Skill layer**：說明 agent 應如何調查、留痕、交付。
2. **Portable product layer**：之後加入 scripts、schemas、templates，讓 Claude Code、OpenCode、一般 CLI 或 web app 可共用。

未來可加入：

- native video model adapter
- local VLM adapter
- browser/computer-use adapter
- comment acquisition adapter
- chart crop helper
- contact sheet helper

這些都是工具，不是思路的主人。
