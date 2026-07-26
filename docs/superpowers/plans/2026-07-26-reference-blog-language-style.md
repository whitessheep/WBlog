# Reference Blog Language Style Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a reusable Chinese language-style guide derived only from `tmp/homurua.github.io/` without prescribing article content or structure.

**Architecture:** Treat the reference site's search index as the complete text corpus and use representative HTML articles when paragraph boundaries or emphasis need confirmation. Distill repeated linguistic patterns into one repository-root guide, separating corpus-wide traits from technical-writing traits, personal-essay traits, and occasional expressions.

**Tech Stack:** Markdown, PowerShell, Hugo-generated HTML/JSON reference corpus, Git.

## Global Constraints

- The only style corpus is `tmp/homurua.github.io/`.
- Files under `content/` are not style evidence and must not influence the guide.
- The guide covers language at sentence and paragraph level only.
- The guide must not prescribe topics, titles, article length, headings, argument structure, explanation order, examples, code blocks, diagrams, summaries, or technical depth.
- Repeated corpus patterns are general traits; isolated phrases are only occasional examples.
- Technical posts and personal essays may have different language intensity while retaining one shared author voice.

---

### Task 1: Build the corpus-backed style guide

**Files:**
- Read: `tmp/homurua.github.io/search/index.json`
- Read: `tmp/homurua.github.io/p/*/index.html`
- Create: `BLOG_STYLE_GUIDE.md`

**Interfaces:**
- Consumes: the scope and evidence rules in `docs/superpowers/specs/2026-07-26-reference-blog-language-style-design.md`
- Produces: a standalone language guide that a future agent can load before drafting or revising an article

- [ ] **Step 1: Confirm the corpus inventory**

Run:

```powershell
$posts = Get-Content -Raw -Encoding UTF8 'tmp\homurua.github.io\search\index.json' | ConvertFrom-Json
"posts=$($posts.Count)"
"chars=$(($posts | ForEach-Object { $_.content.Length } | Measure-Object -Sum).Sum)"
```

Expected output:

```text
posts=39
chars=207745
```

- [ ] **Step 2: Measure recurring expressions before turning them into rules**

Run:

```powershell
$posts = Get-Content -Raw -Encoding UTF8 'tmp\homurua.github.io\search\index.json' | ConvertFrom-Json
$terms = '本篇','可以看到','其实','不过','所以','可能','基本','很简单','需要注意','首先','然后','最后'
foreach ($term in $terms) {
    $occurrences = 0
    $documents = 0
    foreach ($post in $posts) {
        $count = [regex]::Matches($post.content, [regex]::Escape($term)).Count
        $occurrences += $count
        if ($count -gt 0) { $documents++ }
    }
    "${term}: occurrences=$occurrences documents=$documents"
}
```

Expected: `本篇` appears in more than 30 documents; `所以` and `然后` each appear more than 100 times; hedges such as `可能` and `基本` recur across more than 20 documents. Use these results as evidence of connective and certainty habits, not as quotas for future writing.

- [ ] **Step 3: Draft the guide with explicit strength levels**

Create `BLOG_STYLE_GUIDE.md` with these sections and responsibilities:

```markdown
# 博客语言风格指南

## 使用边界
State that only the reference corpus supplies style evidence and that content and structure remain free.

## 一句话概括
Describe the shared voice as an experienced engineer informally整理 and explaining what they understand, with plain language and restrained confidence.

## 稳定特征
Cover tone, vocabulary, connective flow, sentence rhythm, factual voice, uncertainty markers, explanatory stance, rhetoric, and punctuation. Each rule must say what to do, why it matches the corpus, and how to avoid mechanical imitation.

## 技术文章中的语言变化
Describe only language-level changes: lower emotional intensity, direct factual statements, explicit separation of fact and inference, conversational explanations after technical terms, and restrained evaluation.

## 随笔中的语言变化
Describe only language-level changes: stronger first-person presence, colloquial narration, self-deprecating humor, occasional internet expressions, and direct but non-literary emotional statements.

## 偶尔使用，不要复读
List expressions such as hhh, 按下不表, 画图苦手, 开坑/弃坑 as flavor that becomes caricature when repeated.

## 正反例
Provide paired rewrites at sentence or short-paragraph level. Do not prescribe headings or article outlines.

## Agent 可直接使用的提示词
Provide a compact prompt that preserves the writer's content, structure, opinion, and technical depth while applying only the target language voice.

## 语言风格自检
Provide a language-only checklist covering tone, certainty, sentence flow, AI-like polish, catchphrase repetition, and source-boundary compliance.
```

- [ ] **Step 4: Keep examples corpus-faithful without copying passages**

For every positive example, write a new sentence about a neutral technical or everyday subject. Reproduce the linguistic tendencies but do not quote a distinctive sentence or reproduce a paragraph from the reference blog.

Expected: examples demonstrate tone transfer rather than content imitation.

### Task 2: Verify scope, evidence, and usability

**Files:**
- Read: `docs/superpowers/specs/2026-07-26-reference-blog-language-style-design.md`
- Modify: `BLOG_STYLE_GUIDE.md`

**Interfaces:**
- Consumes: the draft guide from Task 1
- Produces: a final guide that constrains language only and can be handed directly to another agent

- [ ] **Step 1: Check that both hard boundaries are explicit**

Run:

```powershell
rg -n 'tmp/homurua\.github\.io|content/|内容|结构|自由|不.*约束' BLOG_STYLE_GUIDE.md
```

Expected: the opening section explicitly identifies the sole corpus, excludes current-blog prose, and states that content and structure remain free.

- [ ] **Step 2: Scan for accidental article-template rules**

Run:

```powershell
rg -n '必须以|开头要|结尾要|固定结构|标题应|章节应|先.*再.*最后|每篇.*总结' BLOG_STYLE_GUIDE.md
```

Expected: no matches that prescribe article organization. If a match occurs inside a negative example, label it clearly; otherwise remove or rewrite it as a language-only rule.

- [ ] **Step 3: Check that traits have strength labels**

Run:

```powershell
rg -n '^## 稳定特征|^## 技术文章中的语言变化|^## 随笔中的语言变化|^## 偶尔使用，不要复读' BLOG_STYLE_GUIDE.md
```

Expected: all four categories are present exactly once.

- [ ] **Step 4: Review for mechanical catchphrase imitation**

Run:

```powershell
$text = Get-Content -Raw -Encoding UTF8 'BLOG_STYLE_GUIDE.md'
$terms = '本篇','可以看到','其实','不过','所以','hhh','按下不表'
foreach ($term in $terms) {
    "${term}: $([regex]::Matches($text, [regex]::Escape($term)).Count)"
}
```

Expected: recurring expressions appear mainly as documented examples. The guide must explicitly warn that they are tendencies rather than required vocabulary.

- [ ] **Step 5: Inspect the final diff**

Run:

```powershell
git diff -- BLOG_STYLE_GUIDE.md
```

Expected: only the new guide is shown; no current article or reference-corpus file is modified.

- [ ] **Step 6: Commit the guide**

Run:

```powershell
git add -- BLOG_STYLE_GUIDE.md
git commit -m "docs: add reference blog language style guide"
```

Expected: one commit containing only `BLOG_STYLE_GUIDE.md`.
