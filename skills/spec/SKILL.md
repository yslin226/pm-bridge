---
name: spec
description: Use when the user receives a requirement from a PM, manager, or stakeholder and wants to clarify or formalize it before starting work. Triggers on /spec, "把這個需求轉成 spec", "PM 給了一個需求", "開工前整理需求", "turn this requirement into a spec".
---

# Requirement-to-Spec Translator

Turn a vague PM requirement into a document with three jobs: **force ambiguities to the surface before coding starts, define testable acceptance criteria both sides agree on, and leave a written record `/pm-doc` can cross-check at delivery time.**

All file paths below are relative to the repository where the user invoked this skill (the target project), NOT the plugin directory.

## Workflow

### 1. Take the requirement in

Input forms, in order of preference: a file path the user gives, pasted text, or the user's verbal description. If the input is only a title (「做一個會員系統」), tell the user the spec will be mostly clarifying questions — that is a valid and useful output, not a failure.

### 2. Restate before expanding

Write the requirement back in 2-3 sentences (需求摘要), preserving the PM's own vocabulary. Do not silently "improve" the requirement — if you think the PM is asking for the wrong thing, that belongs in 待釐清問題, phrased as a question.

### 3. Surface ambiguities — the core value of this skill

Hunt for the gaps that cause rework:

- **Undefined actors**: 「使用者」— which users? Logged-in? Admin? Guest?
- **Undefined boundaries**: empty states, error cases, permission edges, data volume limits
- **Undefined success**: how will the PM decide this is done and working?
- **Hidden dependencies**: other teams, external services, legal/privacy review
- **Unstated non-functional needs**: performance, mobile, i18n, accessibility

Each gap becomes either (a) a stated assumption in the spec, marked 「假設」, or (b) a question in 待釐清問題. Prefer questions for anything that changes scope; prefer assumptions for details you can safely default.

### 4. Draft acceptance criteria

Write criteria as checkboxes, each one **independently verifiable by a non-engineer performing an action and observing a result**:

- ✅ 「未登入使用者點『收藏』時，跳出登入視窗」
- ❌ 「收藏功能運作正常」(not testable)
- ❌ 「API 回傳 200」(not observable by PM)

Every criterion here will be cross-checked by `/pm-doc` at delivery — do not include criteria that cannot be checked against code changes and manual testing.

### 5. Fill the technical outline — outline, not design

技術方案概要 is 3-6 bullet points of approach and touched areas, written for a future engineer (possibly the user in three weeks). It is NOT an implementation plan — no file paths, no schema DDL, no task breakdown. If the user wants a real implementation plan, that is a separate activity after the spec is confirmed.

Estimate (粗估) in ranges only (「2-4 天」), and state what the estimate excludes (review, deploy, QA rounds).

### 6. Produce the document

Fill `templates/requirement-spec.md` (in this skill's directory) **exactly** — same sections, same order. Save to `docs/pm/requirements/<slug>.md` with `<slug>` a short kebab-case feature name; create directories as needed. Set status to `draft`. If the working branch is already known (current branch is a feature branch, or the user names one), fill 對應分支 now — it is the authoritative link `/pm-doc` uses for cross-checking.

Then offer the user a ready-to-send message for the PM containing only 需求摘要 + 待釐清問題, phrased in plain non-technical language — this is the artifact they actually paste into Slack/Line.

When the PM answers, the user re-runs `/spec` with the answers: update the spec in place, resolve questions into assumptions or criteria, and flip status to `confirmed` when 待釐清問題 is empty.

## Writing rules

**Language**: Traditional Chinese (zh-TW) by default. Override order: explicit user request > `pm-bridge` config section in the target repo's CLAUDE.md > language of the input requirement.

**待釐清問題 style**: each question must be answerable by a PM without engineering knowledge, and state why it matters in one clause — 「刪除帳號時，歷史訂單要保留還是一併刪除？（影響資料設計與法規需求）」. Questions the PM cannot understand will simply not get answered.

**No hallucinated constraints**: do not invent stakeholders, compliance requirements, or technical limitations that were not in the input and are not verifiable from the repo.

## What NOT to do

- Do not start implementing. This skill's output is a document, full stop.
- Do not produce an implementation plan, task list, or schedule — out of scope by design.
- Do not resolve scope-changing ambiguities with silent assumptions. Scope questions go to the PM.
- Do not write acceptance criteria for internal quality (test coverage, code style) — those are engineering standards, not PM-verifiable acceptance.
