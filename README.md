# pm-bridge

**工程師 ↔ PM 的雙向翻譯層**，做成 Claude Code plugin。

> A bidirectional translation layer between engineers and PMs, as a Claude Code plugin. `/spec` turns vague PM requirements into technical specs with testable acceptance criteria; `/pm-doc` turns finished work into PM-readable delivery documents, cross-checked against those criteria. Output defaults to Traditional Chinese; configurable.

## 解決什麼問題

- 功能做完，跟 PM 講「重構了 service layer、加了 index」——他聽不懂，也不知道價值在哪。
- 需求進來只有一句「做一個會員系統」，做完才發現理解錯，重工。
- 用 AI 直接寫說明文件，每次格式不一樣、AI 味重、還會編造數字。

pm-bridge 用**固定範本 + 嚴格寫作規則**解決：產出格式每次一致、禁用 AI 慣用廢話、資訊不足時回問而不是瞎編。

## 兩個指令

### `/spec` — 需求進來時

PM 需求（一句話也行）→ 技術規格，包含：

- 需求摘要（保留 PM 原話）
- **可驗收的驗收條件**（每條都是非工程師可操作驗證的）
- **待釐清問題清單**（附「為什麼要問」，可直接貼給 PM）
- 範圍外聲明、粗估

存到你 repo 的 `docs/pm/requirements/`。

### `/pm-doc` — 功能做完時

讀 git diff → PM 看得懂的交付文件，包含：

- 解決了什麼問題、使用者會感覺到什麼改變
- **驗收條件逐條對照**（勾稽 `/spec` 當初寫的條件：完成／部分／未動工）
- 這次沒做的（範圍外）、風險與注意事項

存到你 repo 的 `docs/pm/delivered/`。

### 兩個一起用 = 閉環

```
需求 → /spec（規格+驗收條件）→ 開發 → /pm-doc（交付文件，回頭勾稽驗收條件）
```

所有文件留在 repo 裡，PM 隨時可看，進度有據可查。

## 安裝

尚未發佈到 marketplace。本地試用：clone 本 repo 後以 plugin 開發模式載入：

```bash
claude --plugin-dir /path/to/pm-bridge
```

## 設定

在你的目標 repo 的 CLAUDE.md 加一段（可選）：

```markdown
## pm-bridge
- 輸出語言：zh-TW（或 en、ja...）
```

## Roadmap

- [ ] Phase 1：`/pm-doc` 交付文件產生器（開發中）
- [ ] Phase 2：`/spec` 需求翻譯器 + 驗收條件勾稽
- [ ] Phase 3：GitHub Action 週報——每週自動掃 commits + `docs/pm/`，匯總成主管看的進度週報，發 Slack/email/issue。基於官方 [claude-code-action](https://github.com/anthropics/claude-code-action)，零主機成本

## License

MIT
