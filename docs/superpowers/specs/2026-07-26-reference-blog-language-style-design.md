# Reference Blog Language Style Design

## Goal

Extract a reusable Chinese language-style guide from the reference blog in
`tmp/homurua.github.io/`. The guide will be used by future agents when writing
or revising posts for this blog.

The result should reproduce the reference author's voice at the sentence and
paragraph level without constraining what an article discusses or how it is
organized.

## Source Boundary

The only style corpus is `tmp/homurua.github.io/`, including its technical
articles and personal essays.

Files under `content/` in the current blog are not style evidence. They may be
read only to understand the repository, and no wording, formatting habit, or
article pattern from them should influence the extracted style.

## What The Guide Covers

The guide will describe:

- narrative tone and degree of formality;
- preferred vocabulary and connective expressions;
- sentence length, rhythm, and paragraph-level flow;
- use of first-person statements and uncertainty markers;
- the distinction between factual claims and personal judgment;
- explanatory voice in technical writing;
- emotional intensity and colloquial language in personal writing;
- rhetorical restraint, including the limited use of metaphor, parallelism,
  slogans, and polished quotable lines;
- punctuation and mixed Chinese-English wording where they affect voice;
- positive and negative sentence-level examples;
- a reusable prompt and a language-only self-review checklist for agents.

Technical posts and personal essays will be described separately where their
language differs, while retaining one shared author voice.

## Explicit Non-Goals

The guide must not prescribe:

- topics, opinions, or factual content;
- titles or title formulas;
- article length;
- section hierarchy or heading style;
- argument structure or explanation order;
- the number or placement of examples, code blocks, diagrams, or summaries;
- technical depth.

Those choices remain free for the writer or agent handling each article.

## Evidence Standard

Every claimed style trait should be supported by repeated patterns in the
reference corpus. A memorable phrase from one article is an example, not a
general rule, unless similar usage appears elsewhere.

The guide should distinguish among:

- strong traits that appear throughout the corpus;
- conditional traits associated mainly with technical posts or personal
  essays;
- occasional expressions that can add flavor but should not be imitated too
  frequently.

This prevents surface imitation from turning into repeated catchphrases.

## Deliverable

Create `BLOG_STYLE_GUIDE.md` at the repository root. It should be concise
enough for an agent to load on every writing task, but specific enough to guide
revision decisions.

The document will include the source boundary and non-goals near the top so a
future agent cannot mistake language style for an article template.
