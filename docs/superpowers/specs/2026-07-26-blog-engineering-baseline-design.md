# Blog Engineering Baseline Design

## Goal

Establish a reproducible engineering baseline for the Hugo blog without
changing its visual design, deployment ownership, article titles, publication
dates, or article bodies.

The work will align the Hugo version with the Stack theme, make article URLs
stable, improve basic metadata, document local and Cloudflare Pages setup, and
add a build-only GitHub Actions check.

## Scope

The implementation will:

- set the site language code to `zh-CN`;
- remove the unused example Disqus shortname while leaving comments globally
  disabled;
- require Hugo `0.154.0 extended` as the project baseline;
- add explicit English slugs and Chinese descriptions to every existing post;
- add aliases for every existing post whose public URL changes;
- normalize the `lua` category and tag spelling to `Lua`;
- update the post archetype so new posts include `slug` and `description`;
- ignore `.hugo_build.lock` and `tmp/`;
- add repository setup, preview, build, and Cloudflare Pages instructions;
- add a GitHub Actions workflow that validates the production build but does
  not deploy it.

The implementation will not:

- change the current Stack theme submodule revision;
- change page layout, typography, color, pagination styling, or other visuals;
- change article titles, dates, ordering, prose, code blocks, or diagrams;
- replace the current Cloudflare Pages deployment flow;
- add deployment credentials or require a Cloudflare API token;
- delete or commit the existing untracked `index.txt` file;
- delete files under the existing untracked `tmp/` directory;
- overwrite unrelated working-tree changes.

## Files And Responsibilities

### Site Configuration

`config/_default/hugo.toml` will define `languageCode = "zh-CN"`, retain the
existing Chinese default language and permalink structure, and remove the
unused `[services.disqus]` example configuration.

`config/_default/params.toml` will continue to set
`comments.enabled = false`. The provider value is inert while comments are
disabled and may remain as the theme-compatible default.

### Post Archetype

`archetypes/default.md` will include explicit empty `slug` and `description`
fields. Authors must fill both before publishing a new post. Existing fields
for math, license, visibility, comments, categories, and tags remain intact.

### Existing Post Metadata

Every Markdown file under `content/post/` will receive an explicit English
`slug`. Slugs that already match a short, stable public URL will preserve that
URL. Posts with title-derived Chinese or overly long URLs will move to concise
directory-aligned slugs:

- `Lua 增量 GC` becomes `lua-gc`;
- `Lua 分代 GC` becomes `lua-gc-generation`;
- `Lua 数据类型与GC对象` becomes `lua-struct`;
- `Lua 增量GC 演进：从阈值模型到 Debt (债务) 模型` becomes
  `lua-gc-debt`;
- `RPC 框架` becomes `rpc`.

Before editing, the implementation will capture the current URL list with
`hugo list all`. Each changed URL will be copied exactly into that post's
`aliases` front matter. Hugo will then emit an alias page that redirects the
old URL to the new canonical URL.

Every post will receive a concise Chinese `description` that summarizes only
the existing article. Descriptions must not introduce claims or topics that
are absent from the article body. Existing lowercase `lua` category and tag
values will be normalized to `Lua`; other taxonomy values remain unchanged.

Only front matter may be changed in existing posts. This boundary applies in
particular to the current uncommitted edits in `lua-gc`, `lua-struct`, and
`lua-table`.

## Repository Hygiene

`.gitignore` will add:

```gitignore
.hugo_build.lock
tmp/
```

The already tracked `.hugo_build.lock` file will be removed from Git tracking
without requiring changes to local Hugo behavior. The untracked `tmp/`
directory will stay on disk and become ignored. The untracked `index.txt` file
will remain visible and untouched so the user can decide its future
separately.

## Documentation

A root `README.md` will document:

- the blog's purpose and Hugo/Stack architecture;
- the requirement for Hugo `0.154.0 extended` or newer;
- recursive submodule initialization;
- local preview with drafts;
- production build commands;
- the repository's content and configuration layout;
- the requirement to set Cloudflare Pages `HUGO_VERSION` to `0.154.0` or a
  compatible newer Extended release;
- that deployment remains owned by the existing Cloudflare Pages Git
  integration.

The README will use commands that work from a fresh clone and will explicitly
mention the theme submodule requirement.

## Continuous Integration

`.github/workflows/hugo-check.yml` will run for pushes to `main` and pull
requests. It will:

1. check out the repository with submodules recursively;
2. install Hugo `0.154.0 extended`;
3. run `hugo --gc --minify --panicOnWarning`.

The workflow is validation-only. It will not upload `public/`, deploy to
Cloudflare Pages, use secrets, or alter the existing deployment path.

Warnings are treated as failures so future theme upgrades, deprecated Hugo
configuration, and compatibility drift become visible before deployment.

## Validation

Implementation is complete only when all of the following checks pass:

1. Hugo `0.154.0 extended` completes
   `hugo --gc --minify --panicOnWarning` with exit code 0 and no warnings.
2. `hugo list all` shows an explicit ASCII post URL for every article and no
   title-derived Chinese post URL.
3. Every migrated old URL produces an alias HTML file that redirects to the
   intended new canonical URL.
4. Every post contains a non-empty `slug` and `description`.
5. Category and tag values use `Lua` consistently.
6. A fresh-clone workflow described in the README includes recursive
   submodule initialization and a successful production build path.
7. Git diff inspection confirms that existing article body content and
   unrelated user changes were not altered.
8. The GitHub Actions workflow parses as YAML and uses the same Hugo baseline
   version documented in the README.

If the locally installed Hugo remains below `0.154.0`, validation must use a
temporary compatible Hugo binary rather than weakening the version
requirement or accepting the compatibility warning.

## Success Criteria

- The project has one documented and CI-enforced Hugo compatibility baseline.
- New and existing post URLs are stable and readable.
- Old public URLs continue to resolve through Hugo aliases.
- Basic search and sharing metadata is present for every post.
- A new contributor can clone, initialize, preview, and build the site from
  the root README.
- Build regressions fail before Cloudflare Pages deployment.
- The existing visual design, deployment ownership, article bodies, and user
  working-tree changes remain intact.
