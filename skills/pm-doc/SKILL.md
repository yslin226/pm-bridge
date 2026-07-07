---
name: pm-doc
description: Use when the user has finished a feature, fix, or milestone and wants a document that explains the work to a PM, manager, or other non-technical stakeholder. Reads the git diff, cross-checks acceptance criteria from docs/pm/requirements/, and produces a fixed-template, business-value-oriented delivery document in docs/pm/delivered/. Triggers on /pm-doc, "寫給 PM 的文件", "產交付文件", "跟 PM 說明這次做了什麼", "explain this work to my PM".
---

# PM Delivery Document Generator

Translate engineering work into a document a non-technical PM can read, understand, and forward to their own boss. The output answers three questions PMs actually care about: **what problem is now solved, what will users notice, and how much of the original ask is done.**

All file paths below are relative to the repository where the user invoked this skill (the target project), NOT the plugin directory.

## Workflow

Follow these steps in order. Do not skip the cross-check step.

### 1. Determine the change scope

Resolution order — take the first that applies:

1. The user explicitly specified a range (commits, branch, PR, "today's work") — use it.
2. The current branch differs from the default branch (`main`/`master`) — use `git diff <default>...HEAD` plus the branch's commit messages.
3. There are staged or uncommitted changes — use those.
4. None of the above is decidable — ask the user ONE question offering the concrete options you found. Do not guess.

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
- Numbers only when real (from the diff, tests, or the user's answers).

## What NOT to do

- Do not read Claude Code conversation transcripts (`~/.claude/projects/`) as a data source — they are local-only, noisy, and private. Repo contents and the user's answers are the only sources.
- Do not list files changed, line counts, or commit hashes in the document body — PMs don't care, and it pads the doc.
- Do not editorialize about code quality ("the codebase was messy") — the audience may forward this.
- Do not exceed one page (~400 Chinese characters of body text, excluding the criteria table). If it doesn't fit, the summary is not done yet.
