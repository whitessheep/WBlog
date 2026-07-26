# Writing Homurua Style Skill Design

## Goal

Create a repository-scoped Codex Skill that applies the language voice derived
from the Homurua reference blog when agents write or revise this project's
blog posts.

The Skill becomes the only runtime home of the style guide. After migration,
remove the repository-root `BLOG_STYLE_GUIDE.md` so the rules cannot diverge.

## Location And Name

Create the Skill at:

```text
.agents/skills/writing-homurua-style/
```

Use the name `writing-homurua-style`. Repository-scoped Skills live under
`.agents/skills/`, allowing Codex sessions launched inside this repository to
discover the Skill.

## Trigger Scope

The Skill should trigger when an agent:

- writes a new blog post for this repository;
- revises, rewrites, or polishes an existing blog post, especially under
  `content/post/`;
- is asked to use the Homurua style, the reference-blog style, or this blog's
  established language voice.

The Skill may also be invoked explicitly as `$writing-homurua-style`.

## File Structure

```text
.agents/skills/writing-homurua-style/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── style-guide.md
```

`SKILL.md` contains only the workflow, hard boundaries, reference-loading rule,
and final self-check. It stays concise so triggering the Skill does not load
the full language guide immediately.

`references/style-guide.md` contains the complete content currently stored in
`BLOG_STYLE_GUIDE.md`: shared voice, stable language traits, technical-writing
and personal-essay variations, occasional expressions, examples, reusable
prompt guidance, self-review rules, and corpus evidence.

`agents/openai.yaml` contains generated UI metadata matching the final Skill.

No scripts or assets are needed. The work is judgment-based language guidance,
and no repeated deterministic transformation requires a helper script.

## Runtime Behavior

When triggered, the Skill must:

1. Read `references/style-guide.md` completely before drafting or revising.
2. Treat the style rules as language-only constraints.
3. Preserve user-defined content, opinions, title, structure, length,
   technical depth, and argument order.
4. For revisions, preserve the existing article's decided content and
   organization.
5. For new writing, follow the user's brief without allowing the style guide
   to invent article requirements.
6. Select the technical or personal language intensity from the actual text;
   mixed articles may vary by paragraph.
7. Run the language-only self-check before returning or saving the result.

The Skill should reproduce the voice rather than surface catchphrases. Terms
such as `hhh`, “其实”, or “可以看到” are optional evidence of style, not
required vocabulary.

## Source Boundary

The migrated guide must retain these facts:

- its language model was derived only from `tmp/homurua.github.io/`;
- prose under the current blog's `content/` directory was not style evidence;
- future use does not require reading the `tmp/` reference copy.

This makes the Skill self-contained while preserving the provenance and
preventing current generated prose from feeding back into the target style.

## Migration

Copy the complete guide into
`.agents/skills/writing-homurua-style/references/style-guide.md`, adjusting only
paths or wording required for its new location. Verify that no substantive
language rule, example, prompt rule, self-check item, or evidence note was
lost.

After the Skill passes validation and forward tests, delete
`BLOG_STYLE_GUIDE.md`. Do not leave a second copy or a redirect file at the
repository root.

Existing blog articles, reference-site files, and unrelated working-tree
changes must not be modified or staged.

## Validation Strategy

Follow skill TDD:

1. Run fresh-agent baseline scenarios without the Skill.
2. Record default failures such as AI-style promotional wording, excessive
   literary polish, uncertainty stated as fact, catchphrase stuffing, or style
   guidance changing content and structure.
3. Initialize and write the minimum Skill that addresses observed failures.
4. Run the same scenarios with `$writing-homurua-style` loaded.
5. Compare outputs for language fit and preservation of the supplied content
   and organization.
6. Refine only when tests reveal a real gap.

Use at least:

- one technical-source explanation scenario;
- one personal-essay revision scenario;
- one boundary scenario that explicitly supplies a structure the Skill must
  not change.

Run `quick_validate.py` on the final Skill folder. Check that `SKILL.md` has
valid frontmatter, the folder name matches the Skill name, references resolve,
`agents/openai.yaml` matches the Skill, and only the Skill plus removal of the
old root guide are included in the migration commit.

## Success Criteria

- Codex discovers the project Skill from `.agents/skills/`.
- The Skill triggers for blog writing, editing, polishing, and named-style
  requests.
- Technical writing remains plain, direct, conversational, and careful about
  certainty.
- Personal writing becomes more colloquial without turning literary or
  stacking internet slang.
- The Skill never prescribes article content or organization.
- The complete style guide exists only inside the Skill.
- Baseline-versus-Skill tests demonstrate a material improvement.
- Official Skill validation passes.
