# Writing Homurua Style Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create and validate the repository-scoped `writing-homurua-style` Codex Skill, move the complete language guide into it, and remove the old root guide.

**Architecture:** Store concise runtime instructions in `SKILL.md` and the complete language reference in `references/style-guide.md`. Generate `agents/openai.yaml` with the official Skill Creator utility, then use fresh-agent baseline and forward tests to verify that the Skill changes language voice without changing content or structure.

**Tech Stack:** Markdown, YAML, Codex Agent Skills, Python Skill Creator utilities, Git, Codex subagents for forward testing.

## Global Constraints

- Create the project Skill at `.agents/skills/writing-homurua-style/`.
- Use the exact Skill name `writing-homurua-style`.
- The Skill must trigger for new blog writing, article revision or polishing, `content/post/` work, Homurua-style requests, reference-blog-style requests, and established-project-voice requests.
- Style rules control language only; content, opinions, title, structure, length, technical depth, and argument order remain user-controlled.
- Preserve the complete substantive content of `BLOG_STYLE_GUIDE.md` inside `references/style-guide.md`.
- Keep `tmp/homurua.github.io/` as provenance only; the final Skill must not require it at runtime.
- Do not use current `content/` prose as style evidence.
- Do not modify or stage existing blog article edits, `index.txt`, or `tmp/`.
- Delete `BLOG_STYLE_GUIDE.md` only after validation and forward tests pass.
- Include no README, changelog, test log, script, asset, or other auxiliary file in the Skill.

---

### Task 1: Run RED baseline scenarios without the Skill

**Files:**
- Read: `docs/superpowers/specs/2026-07-26-writing-homurua-style-skill-design.md`
- Read: `BLOG_STYLE_GUIDE.md`
- Do not create repository files

**Interfaces:**
- Consumes: three fixed user prompts with no access to the future Skill
- Produces: raw baseline outputs and a concise list of observed language failures for Task 3

- [ ] **Step 1: Launch three fresh baseline agents with no conversation context and no Skill reference**

Use `fork_turns="none"`. Give each agent exactly one of the following prompts.

Technical scenario:

```text
请根据下面的信息写一段中文技术博客正文。信息和顺序不能改变，不要增加总结，也不要改动两个二级标题。

## 入队
worker queue 使用环形数组。enqueue 会把任务指针写入 tail 指向的位置，然后移动 tail。

## 出队
dequeue 从 head 指向的位置取出任务，然后移动 head。代码里没有锁。已知这里只有一个生产者和一个消费者，但源码没有说明这是不是不加锁的设计原因。
```

Personal scenario:

```text
请润色下面四句话，只调整语言，不改变句子顺序、事实或情绪强度：

博客两个月没有更新。最近工作比较忙，休息的时候基本都在打游戏。中间想过三次要写点东西，但是一次都没有开始。现在准备恢复更新。
```

Boundary scenario:

```text
请把下面内容改成自然的中文博客语言。必须保留标题、两个小节、段落顺序和“这个优化不值得做”的结论，不得增加引言或总结。

# 缓存优化记录

## 现象
一次查询平均需要 4 微秒，缓存后变成 3.8 微秒。

## 判断
缓存增加了失效处理和并发控制。我认为这个优化不值得做。
```

- [ ] **Step 2: Record the raw outputs before inspecting the style guide**

Keep the outputs in the agent messages only. Do not create eval files inside the Skill or repository.

Expected: each agent returns a usable article fragment, but at least one output exhibits a default-model style mismatch such as promotional emphasis, literary polish, formal AI transitions, excessive headings, strengthened certainty, or changed structure.

- [ ] **Step 3: Score each baseline against the fixed rubric**

Use one point for each satisfied criterion:

```text
1. Preserves every supplied fact and opinion.
2. Preserves the supplied structure and order.
3. Uses plain conversational written Chinese.
4. Avoids AI/media phrases such as 深入探索、核心枢纽、显而易见、综上所述、设计哲学.
5. Keeps confirmed facts direct and uncertain intent explicitly uncertain.
6. Does not force catchphrases such as 其实、可以看到、本篇、hhh.
7. Technical text stays restrained; personal text stays colloquial without literary expansion.
```

Expected: document exact failures, not inferred intentions. These failures determine the minimum instructions written in Task 3.

### Task 2: Initialize the project Skill

**Files:**
- Create: `.agents/skills/writing-homurua-style/SKILL.md`
- Create: `.agents/skills/writing-homurua-style/agents/openai.yaml`
- Create directory: `.agents/skills/writing-homurua-style/references/`

**Interfaces:**
- Consumes: Skill name, project path, and UI metadata below
- Produces: an initialized Skill directory with valid generated metadata and temporary `SKILL.md` template content to replace in Task 3

- [ ] **Step 1: Initialize with the official Skill Creator utility**

Run:

```powershell
python 'C:\Users\WhiteSheep\.codex\skills\.system\skill-creator\scripts\init_skill.py' writing-homurua-style `
  --path 'F:\my_project\myblog\.agents\skills' `
  --resources references `
  --interface 'display_name=Homurua 博客文风' `
  --interface 'short_description=用既定的平实工程师声线写作或润色中文博客文章' `
  --interface 'default_prompt=Use $writing-homurua-style to write or revise this blog post in the project language voice.'
```

Expected: the utility creates the exact Skill directory, `SKILL.md`, `agents/openai.yaml`, and `references/` without scripts, assets, or examples.

- [ ] **Step 2: Inspect generated files before editing**

Run:

```powershell
Get-ChildItem '.agents\skills\writing-homurua-style' -Recurse -Force
Get-Content -Raw -Encoding UTF8 '.agents\skills\writing-homurua-style\SKILL.md'
Get-Content -Raw -Encoding UTF8 '.agents\skills\writing-homurua-style\agents\openai.yaml'
```

Expected: generated metadata uses quoted string values, the default prompt explicitly names `$writing-homurua-style`, and no unexpected resource directory exists.

### Task 3: Write the minimum Skill and migrate the full reference

**Files:**
- Modify: `.agents/skills/writing-homurua-style/SKILL.md`
- Create: `.agents/skills/writing-homurua-style/references/style-guide.md`
- Read: `BLOG_STYLE_GUIDE.md`

**Interfaces:**
- Consumes: baseline failures from Task 1 and the complete root language guide
- Produces: the runtime Skill instructions and its self-contained detailed reference

- [ ] **Step 1: Replace the generated SKILL.md with the runtime contract**

Write this structure, adjusting only language needed to address an observed baseline failure:

```markdown
---
name: writing-homurua-style
description: Use when writing, revising, rewriting, or polishing Chinese blog posts in this repository, especially content/post articles, or when the user asks for the Homurua, reference-blog, or established project language style.
---

# Writing Homurua Style

## Core Principle

Apply the Homurua-derived voice to language only. Keep the user's content,
opinions, title, structure, length, technical depth, and argument order intact.

## Required Workflow

1. Read `references/style-guide.md` completely before drafting or revising.
2. Determine whether each passage is technical, personal, or mixed from the
   supplied content. Do not invent a category-driven article structure.
3. Preserve all user-defined requirements. For revisions, preserve existing
   facts, conclusions, headings, and paragraph order unless the user asks to
   change them.
4. Draft or revise with the language contract below.
5. Run the reference guide's language-only self-check before returning or
   saving the article.

## Output Contract

| Text type | Required voice |
|---|---|
| All writing | Plain, direct, conversational written Chinese; simple causal transitions; restrained evaluation; no forced catchphrases |
| Technical | State confirmed behavior directly, mark inference as inference, explain identifiers in ordinary Chinese, keep emotional intensity low |
| Personal | Use a stronger first-person and colloquial voice, allow light self-deprecation, keep emotion concrete and non-literary |

For new writing, take content and organization from the user's brief. For
revision, return the same article with language changed only.

## Quick Reference

- Prefer ordinary transitions such as “所以”“不过”“然后”“就是” when they
  fit naturally.
- Keep repeated technical names when repetition improves clarity.
- Mark uncertain motives with “可能”“应该”“个人感觉” or an equivalent hedge.
- Keep internet slang occasional and contextual.
- Preserve mixed Chinese-English spacing and code identifiers.

## Common Mistakes

- Treating the style guide as an article template.
- Rewriting user opinions or strengthening uncertain claims.
- Adding introductions, summaries, headings, metaphors, or lessons the user did
  not request.
- Repeating “其实”“可以看到”“本篇” or `hhh` to simulate the voice.
- Using current `content/` prose as style evidence.
- Producing literary, promotional, textbook, or knowledge-influencer language.
```

- [ ] **Step 2: Move the full style guide into the Skill reference**

Use `apply_patch` to create
`.agents/skills/writing-homurua-style/references/style-guide.md` with the full
substantive content of `BLOG_STYLE_GUIDE.md`. Retain the provenance statement,
stable traits, technical and personal variations, occasional expressions,
positive and negative examples, reusable prompt, self-check, and corpus
evidence.

Change the opening wording only as needed to make the reference self-contained
inside the Skill. Do not shorten or summarize the guide during migration.

- [ ] **Step 3: Verify migration completeness before deleting the source**

Run:

```powershell
$old = Get-Content -Raw -Encoding UTF8 'BLOG_STYLE_GUIDE.md'
$new = Get-Content -Raw -Encoding UTF8 '.agents\skills\writing-homurua-style\references\style-guide.md'
$headings = @('## 使用边界','## 一句话概括','## 稳定特征','## 技术文章中的语言变化','## 随笔中的语言变化','## 偶尔使用，不要复读','## 正反例','## Agent 可直接使用的提示词','## 语言风格自检','## 提炼依据')
foreach ($heading in $headings) {
  if (-not $new.Contains($heading)) { throw "Missing migrated section: $heading" }
}
if ($new.Length -lt [int]($old.Length * 0.95)) { throw 'Migrated guide is unexpectedly shorter' }
"migration_check=passed old_chars=$($old.Length) new_chars=$($new.Length)"
```

Expected: all ten sections exist and the migrated guide retains at least 95%
of the source character count.

### Task 4: Validate the Skill structure

**Files:**
- Read: `.agents/skills/writing-homurua-style/SKILL.md`
- Read: `.agents/skills/writing-homurua-style/agents/openai.yaml`
- Read: `.agents/skills/writing-homurua-style/references/style-guide.md`

**Interfaces:**
- Consumes: completed Skill files from Task 3
- Produces: structural validation evidence required before forward testing

- [ ] **Step 1: Run official quick validation**

Run:

```powershell
python -X utf8 'C:\Users\WhiteSheep\.codex\skills\.system\skill-creator\scripts\quick_validate.py' '.agents\skills\writing-homurua-style'
```

Expected: validation succeeds with no frontmatter, naming, or folder errors.

- [ ] **Step 2: Run project-specific assertions**

Run:

```powershell
$skill = Get-Content -Raw -Encoding UTF8 '.agents\skills\writing-homurua-style\SKILL.md'
$yaml = Get-Content -Raw -Encoding UTF8 '.agents\skills\writing-homurua-style\agents\openai.yaml'
$guide = Get-Content -Raw -Encoding UTF8 '.agents\skills\writing-homurua-style\references\style-guide.md'
if ($skill -notmatch '^---\r?\nname: writing-homurua-style\r?\ndescription: Use when') { throw 'Invalid Skill frontmatter' }
if (-not $skill.Contains('Read `references/style-guide.md` completely')) { throw 'Missing reference loading rule' }
if ($skill -notmatch "Keep the user's content,\s+opinions, title, structure, length") { throw 'Missing language-only boundary' }
if (-not $yaml.Contains('$writing-homurua-style')) { throw 'Default prompt does not name the Skill' }
if (-not $guide.Contains('tmp/homurua.github.io/')) { throw 'Missing corpus provenance' }
"project_checks=passed"
```

Expected: `project_checks=passed`.

### Task 5: Run GREEN forward tests with the Skill

**Files:**
- Read: `.agents/skills/writing-homurua-style/SKILL.md`
- Read: `.agents/skills/writing-homurua-style/references/style-guide.md`
- Do not create repository files

**Interfaces:**
- Consumes: the same three prompts from Task 1 plus the completed Skill
- Produces: raw forward-test outputs and rubric scores demonstrating whether the Skill corrects baseline failures

- [ ] **Step 1: Launch three fresh agents with no conversation history**

Use `fork_turns="none"`. Prefix each Task 1 prompt with:

```text
Use $writing-homurua-style at F:\my_project\myblog\.agents\skills\writing-homurua-style to complete this task.
```

Do not include expected answers, baseline diagnoses, or the rubric in the agent
prompt.

- [ ] **Step 2: Score outputs with the same seven-point rubric**

Expected for every scenario:

```text
- All supplied facts and opinions preserved.
- Supplied structure and order preserved.
- No unrequested introduction, summary, or heading.
- No promotional, literary, textbook, or AI-template phrasing.
- Technical uncertainty remains explicitly uncertain.
- No catchphrase stacking.
- Technical and personal voice differ appropriately.
```

- [ ] **Step 3: Refactor only if a real failure remains**

If a forward test fails, edit only the smallest relevant instruction in
`SKILL.md` or `references/style-guide.md`, rerun `quick_validate.py`, and repeat
the failed scenario with a fresh agent.

Expected: all three scenarios satisfy all seven rubric criteria. Because this
is a behavior-shaping Skill rather than a discipline-enforcement Skill, do not
add a rationalization table, red-flags section, or flowchart unless testing
reveals that one is necessary.

### Task 6: Remove the old guide and verify the final migration

**Files:**
- Delete: `BLOG_STYLE_GUIDE.md`
- Verify: `.agents/skills/writing-homurua-style/SKILL.md`
- Verify: `.agents/skills/writing-homurua-style/agents/openai.yaml`
- Verify: `.agents/skills/writing-homurua-style/references/style-guide.md`

**Interfaces:**
- Consumes: a validated and forward-tested Skill
- Produces: one canonical style guide inside the project Skill

- [ ] **Step 1: Delete the root guide with apply_patch**

Delete `BLOG_STYLE_GUIDE.md` only after Tasks 4 and 5 pass.

- [ ] **Step 2: Verify there is one runtime copy**

Run:

```powershell
if (Test-Path 'BLOG_STYLE_GUIDE.md') { throw 'Root guide still exists' }
$matches = @(rg -l '^# 博客语言风格指南$' '.agents\skills\writing-homurua-style' -g '*.md')
if ($matches.Count -ne 1 -or $matches[0] -notmatch 'references[\\/]style-guide\.md$') { throw "Unexpected guide copies: $($matches -join ', ')" }
"canonical_guide=$($matches[0])"
```

Expected: exactly one guide at
`.agents/skills/writing-homurua-style/references/style-guide.md`.

- [ ] **Step 3: Verify the final diff scope**

Run:

```powershell
git status --short
git diff --check
git diff -- BLOG_STYLE_GUIDE.md '.agents/skills/writing-homurua-style'
```

Expected: implementation changes consist of the new Skill directory and the
deleted root guide. Existing user changes under `content/post/`, `index.txt`,
and `tmp/` remain unstaged and untouched.

- [ ] **Step 4: Commit only the Skill migration**

Run:

```powershell
git add -- '.agents/skills/writing-homurua-style' 'BLOG_STYLE_GUIDE.md'
git diff --cached --check
git diff --cached --name-status
git commit -m "feat: add project blog style skill"
```

Expected: the commit contains the new Skill files and deletion of
`BLOG_STYLE_GUIDE.md`, with no blog article, `tmp/`, or `index.txt` change.
