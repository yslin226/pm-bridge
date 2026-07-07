# pm-bridge

Claude Code plugin：工程師 ↔ PM 的雙向翻譯層。`/spec` 把 PM 需求轉成含驗收條件的技術規格；`/pm-doc` 把做完的工作轉成 PM 看得懂的交付文件，並回頭勾稽驗收條件。Phase 3 會加 GitHub Action 週報。

## 專案結構

```
.claude-plugin/plugin.json     # plugin manifest
skills/
  pm-doc/SKILL.md              # 交付文件產生器
  pm-doc/templates/delivery-doc.md
  spec/SKILL.md                # 需求→規格翻譯器
  spec/templates/requirement-spec.md
docs/superpowers/specs/        # 本專案自己的設計文件
```

注意區分兩層：SKILL.md 裡寫的 `docs/pm/...` 路徑是指「使用者的目標 repo」，不是本 repo。本 repo 是 plugin 原始碼。

## 慣例

- SKILL.md 本文用英文寫（社群慣例、觸發可靠度），skill 的「輸出」預設繁體中文——這是設計決策，見 docs/superpowers/specs/。
- 範本檔（templates/*.md）是產品的核心資產：固定章節、固定順序，skill 指示 Claude 不得增刪章節。改範本 = 改產品行為，要謹慎。
- 兩個 skill 的共同鐵律：不編造數字、不讀 `~/.claude/projects/` 對話紀錄、只依賴 repo 內容與使用者回答。改 SKILL.md 時不要弱化這些規則。
- 面向使用者的文件（README、範本）用繁體中文；程式碼與 commit message 用英文。

## 測試

手動測試：在任一測試 repo 裝本 plugin（`claude plugin install` 指向本地路徑或 marketplace），實跑 `/spec`、`/pm-doc`，檢查輸出是否嚴格符合範本。尚無自動化測試（Phase 3 後考慮以 headless `claude -p` 做煙霧測試）。

## Roadmap

- Phase 1（現在）：/pm-doc
- Phase 2：/spec + 兩者勾稽
- Phase 3：GitHub Action 週報（掃 commits + docs/pm/ → 匯總，發 Slack/email/issue）
