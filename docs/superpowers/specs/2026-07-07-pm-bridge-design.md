# pm-bridge 設計文件

- 日期：2026-07-07
- 狀態：已與需求方（yslin226）確認方向與三階段範圍

## 1. 問題與目標

工程師與 PM 之間存在雙向翻譯斷層：

1. **工程 → PM**：功能做完，技術性的說明 PM 聽不懂、看不出價值；用裸 AI 產文件則格式不一致、AI 味重、會編造內容。
2. **PM → 工程**：需求進來模糊不清，做完才發現理解錯，重工。
3. **進度追蹤**：主管追進度，工程師懶得寫週報，現有工具只看 commit message 猜，品質差。

目標：一個開源 Claude Code plugin（+ 後期 GitHub Action），讓上述三件事自動化且品質穩定。專案定位為開源貢獻 + 求職作品集。

## 2. 競品調研結論（2026-07-07）

- 工程→PM 翻譯：有近似品（aakashg/pm-claude-skills 的 Status Update Writer、各式 changelog skill），但「讀 diff → 價值導向文件 + 固定範本 + 繁中」的切角無人做。
- 需求→spec：紅海（GitHub spec-kit、BMAD、plan mode），單獨做打不過；但作為「驗收條件供交付端勾稽」的配套有差異化。
- commit 週報：紅海（gitmore.io、ai-commit-report-generator-cli、gitgossip）；差異化靠「有 /spec 與 /pm-doc 留下的高品質素材」而非只看 commit message。
- 名稱 `pm-bridge` 於 GitHub 無撞名（查證日 2026-07-07）。

## 3. 架構

一個 plugin repo，三個交付物，分三期：

```
[Phase 2] /spec ────────► docs/pm/requirements/<slug>.md（規格+驗收條件）
                                      │ 勾稽
[Phase 1] /pm-doc ──讀 git diff──────┴─► docs/pm/delivered/<date>-<slug>.md
[Phase 3] GitHub Action（cron）─讀 commits + docs/pm/─► 週報 → Slack/email/issue
```

關鍵設計決策：

| 決策 | 選擇 | 理由 |
|---|---|---|
| 包裝形式 | plugin（內含多個 skill） | 多能力一起發佈、marketplace 可安裝可更新 |
| 資料來源 | 只用 repo 內容 + 使用者回答 | 對話紀錄（~/.claude/projects/）本機限定、噪音大、有隱私問題；Action 端也讀不到 |
| 資訊不足時 | 回問使用者（每次最多 2 題） | 「不編造」是產品核心承諾，也是去 AI 味關鍵 |
| 產出一致性 | 固定範本檔 + SKILL.md 禁止增刪章節 | 解決裸 AI 每次格式不同的痛點 |
| 輸出語言 | 預設 zh-TW，可由目標 repo CLAUDE.md 覆寫 | 主要受眾台灣工程圈，但不鎖死 |
| SKILL.md 語言 | 英文 | 社群慣例與觸發可靠度；輸出語言另行控制 |
| 報告端形態 | GitHub Action（非 CLI、非 SaaS） | 零安裝、零主機；LLM 費用由使用者的 API key secret 承擔 |

## 4. 元件規格

### 4.1 /pm-doc（Phase 1）

- 輸入：git diff（範圍解析順序：使用者指定 > branch vs default > staged；不可判定則問一題）
- 勾稽：在 docs/pm/requirements/ 找對應規格（branch 名 > commit 引用 > 內容相似），逐條標記驗收條件狀態並回寫
- 輸出：依 templates/delivery-doc.md，存 docs/pm/delivered/YYYY-MM-DD-<slug>.md
- 寫作規則：禁 hype 形容詞與 AI 填充語、術語首次出現附白話解釋、正文一頁內
- 錯誤處理：無對應規格 → 照常產出並註明；資訊不足 → 回問（上限 2 題），使用者不知道則寫「尚未量測」

### 4.2 /spec（Phase 2）

- 輸入：檔案路徑 / 貼上文字 / 口述；只有一句話也是合法輸入（產出以待釐清問題為主）
- 核心步驟：複述需求（保留 PM 用語）→ 挖模糊點（角色、邊界、成功定義、依賴、非功能需求）→ 每個模糊點變「假設」或「回問」
- 驗收條件標準：每條可由非工程師操作驗證；此為 /pm-doc 勾稽的基準
- 輸出：依 templates/requirement-spec.md，存 docs/pm/requirements/<slug>.md，status: draft；另產可直接貼給 PM 的回問訊息
- 迭代：PM 回覆後重跑 /spec 就地更新，問題清空後 status: confirmed
- 明確不做：實作計畫、任務拆解、排程

### 4.3 GitHub Action 週報（Phase 3，屆時另立設計文件）

- cron 排程，讀該週 commits + docs/pm/ 增量，匯總成週報
- 輸出通道：Slack webhook / email / GitHub issue（擇一或多）
- API key 由使用者以 Action secret 提供
- 差異化：能報「需求 X 驗收條件完成 3/5」而非只有 commit 摘要

## 5. 測試策略

- Phase 1-2：手動驗證——本地安裝 plugin，於測試 repo 實跑，檢查（a）觸發正確（b）輸出嚴格符合範本（c）禁則未違反（無編造數字、無 AI 填充語）
- Phase 3 前：評估以 headless `claude -p --output-format json` 做自動化煙霧測試
- 驗收即範本：範本檔本身就是輸出的驗收基準

## 6. 風險與已知限制

- 去 AI 味可大幅改善、無法保證 100%——以固定範本、禁詞表、回問機制逼近
- skill 類專案護城河低，價值在痛點切角與範本品質，接受此定位（開源+作品集導向）
- Anthropic 或社群後續可能推出重疊功能，屆時調整差異化重點（勾稽閉環是最難被單一 skill 取代的部分）

## 7. 重用策略（不死刻原則）

能引用現成的就引用，自己只做差異化的部分：

- **Phase 3 基底**：不自寫 CI 整合，直接建立在官方 [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) 之上——它已處理好「在 GitHub Actions 跑 Claude + API key secret」整段；我們只提供 prompt + 範本 + workflow 範例檔。Phase 3 工作量預期由 2 週降至數天。
- **參考不重造**：驗收條件寫法參考既有 BDD/Gherkin 慣例；changelog 分類參考 Keep a Changelog；週報輸出格式參考 gitmore / ai-commit-report-generator-cli 的報告結構（只參考結構，內容管線不同）。
- **skill 撰寫慣例**：遵循 superpowers 的 writing-skills 規範與社群 SKILL.md 格式，不自創格式。
- 引用他人程式碼時遵守其授權條款並於 README 註明出處。

## 8. 里程碑

1. Phase 1（~1 週）：/pm-doc 可用、本地安裝驗證通過 → 可發佈
2. Phase 2（~1 週）：/spec + 勾稽 → 更新 README、發佈 marketplace
3. Phase 3（~2 週）：GitHub Action → 另立設計文件後實作
