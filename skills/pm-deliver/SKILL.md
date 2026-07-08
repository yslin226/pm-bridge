---
name: pm-deliver
description: Use when the user has finished a feature, fix, or milestone and wants to explain the work to a PM, manager, or other non-technical stakeholder, or asks for a delivery document. Triggers on /pm-deliver, "寫給 PM 的文件", "產交付文件", "跟 PM 說明這次做了什麼", "explain this work to my PM".
---

# PM Delivery Document Generator

Translate engineering work into a document a non-technical PM can read, understand, and forward to their own boss. The output answers three questions PMs actually care about: **what problem is now solved, what will users notice, and how much of the original ask is done.**

All file paths below are relative to the repository where the user invoked this skill (the target project), NOT the plugin directory.

## Workflow

Follow these steps in order. Do not skip the cross-check step.

### 1. Determine the change scope

Resolution order — take the first that applies:

1. The user explicitly specified a range (commits, branch, PR, "today's work") — use it.
2. A default branch (`main` or `master`) exists AND the current branch differs from it — use `git diff <default>...HEAD` plus the branch's commit messages.
3. There are staged or uncommitted changes — use those.
4. No default branch exists, or the user is already on it (single-branch repo, or committing straight to `main`/`master`) — use the most recent commit(s) since the last `/pm-deliver` run, or if none is known, the last commit on the branch.
5. None of the above is decidable — ask the user ONE question offering the concrete options you found. Do not guess.

Before relying on step 2, verify the default branch actually exists (`git rev-parse --verify main` / `master`) — do not assume one is present.

### 2. Read the changes

Read the diff AND enough surrounding code to understand user-visible behavior. You are looking for:

- User-facing changes (UI, API responses, error messages, performance)
- Behavior changes (what happens differently than before)
- Deliberately excluded scope (TODOs, follow-up markers, feature flags left off)

Ignore pure refactors, formatting, and test-only changes when describing value — but note test coverage as a quality signal.

### 3. Cross-check the requirement spec

Look in `docs/pm/requirements/` for a spec matching this work. Match by, in order: the spec's 對應分支 field equal to the current branch (authoritative), branch name ↔ spec slug, commit message references, content similarity. If only fuzzy matching succeeded, confirm the match with the user before cross-checking, and write the branch name into the spec's 對應分支 field so future runs are deterministic. 

- **Match found**: load its acceptance criteria (驗收條件). For each criterion, determine from the diff whether it is 完成 / 部分完成 / 未動工. Update the checkboxes in the requirement file itself and set its status field if all criteria pass.
- **No match**: proceed without it, and note in the output that no requirement spec was linked. Never fabricate criteria.

### 4. Fill information gaps — ask, never invent

If the diff cannot tell you something the template needs (measured performance numbers, why a scope cut was made, rollout date), ask the user. **At most TWO questions, in one message.** If the user does not know, write 「尚未量測」or omit the claim entirely.

Hard rule: **no invented numbers, no invented user quotes, no speculative benefits.** A delivery doc that says "faster" without a number is honest; one that says "40% faster" without a measurement is a lie the PM will repeat upward.

### 5. Produce the document

Fill `templates/delivery-doc.md` (in this skill's directory) **exactly** — same sections, same order, no additions or deletions. Empty sections get 「無」, not deletion.

Save to `docs/pm/delivered/YYYY-MM-DD-<slug>.md` where `<slug>` is a short kebab-case feature name. Create directories as needed. Show the user the full document and where it was saved.

After saving, remind the user (one line) that raw `.md` reads as plain text with visible `##`/`**` syntax — for sharing with a non-technical PM, open it in a Markdown preview instead (GitHub/GitLab web view, VS Code preview, `glow <file>`), or ask this skill to also produce an HTML version (see below).

### 6. Optional: HTML export

If the user asks for an HTML copy (or a PM has previously said the raw file "looks AI-written"), render the saved `.md` to a single self-contained HTML file next to it (same name, `.html` extension) using `pandoc <file>.md -o <file>.html --standalone --metadata title="<功能名稱>"` if `pandoc` is available; otherwise tell the user it is not installed rather than hand-rolling a converter. The HTML file is a **rendering of the Markdown, never a replacement for it** — the `.md` stays the source of truth `pm-deliver`/`pm-translate` cross-check against.

## Writing rules

These rules exist because generic AI output is verbose, hype-flavored, and inconsistent. Violating them defeats the purpose of this skill.

**Language**: Traditional Chinese (zh-TW) by default. Override order: explicit user request > `pm-bridge` config section in the target repo's CLAUDE.md > language of the matched requirement spec.

**Audience**: a PM who cannot read code and will forward this document to their own manager. Every sentence must survive that second hop.

**Style**:
- Plain declarative sentences. One idea per sentence.
- Concrete before abstract: 「結帳頁少填 3 個欄位」beats 「優化了使用者體驗」.
- Technical terms allowed only when unavoidable, and each gets a one-phrase plain explanation in parentheses on first use.
- No hype adjectives: 強大、大幅、顯著、完美、無縫 are banned unless backed by a number in the same sentence.
- Banned filler phrases: 值得注意的是、總的來說、綜上所述、在當今…的時代、首先/其次/最後 as paragraph scaffolding.
- Prefer everyday verbs over bureaucratic ones: 做了/改了/加了/修好了 over 進行了/予以/呈現/實現. Write as if explaining to a colleague over coffee, not filing a report.
- Numbers only when real (from the diff, tests, or the user's answers).

## What NOT to do

- Do not read Claude Code conversation transcripts (`~/.claude/projects/`) as a data source — they are local-only, noisy, and private. Repo contents and the user's answers are the only sources.
- Do not list files changed, line counts, or commit hashes in the document body — PMs don't care, and it pads the doc.
- Do not editorialize about code quality ("the codebase was messy") — the audience may forward this.
- Do not exceed one page (~400 Chinese characters of body text, excluding the criteria table). If it doesn't fit, the summary is not done yet.
