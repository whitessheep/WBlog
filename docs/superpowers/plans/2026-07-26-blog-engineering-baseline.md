# Blog Engineering Baseline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the Hugo blog reproducible, URL-stable, documented, and build-checked without changing its visual design, deployment ownership, or article bodies.

**Architecture:** Keep the existing Hugo and Stack theme structure. Harden repository configuration, migrate only post front matter, document the build contract, and add a validation-only GitHub Actions workflow pinned to Hugo Extended 0.154.0.

**Tech Stack:** Hugo Extended 0.154.0, TOML, YAML front matter, Markdown, GitHub Actions, Cloudflare Pages.

## Global Constraints

- Work in the current checkout only after explicit user consent because `content/post/lua-gc/index.md`, `content/post/lua-struct/index.md`, and `content/post/lua-table/index.md` already contain user-owned working-tree changes that must remain present.
- Do not change the Stack theme submodule revision.
- Do not change page visuals, article titles, dates, ordering, prose, headings, code blocks, or diagrams.
- Existing post edits may touch front matter only.
- Preserve the existing Cloudflare Pages Git deployment; CI validates but never deploys.
- Require Hugo `0.154.0 extended` as the documented and CI-enforced baseline.
- Do not delete, stage, or commit the existing untracked `index.txt` file.
- Do not delete files under `tmp/`; add `tmp/` to `.gitignore` only.
- Normalize only lowercase `lua` taxonomy values to `Lua`; leave other taxonomy values unchanged.
- Apply the repository `writing-homurua-style` language rules only to new Chinese descriptions.
- Before every commit, inspect `git diff --cached` and remove any unrelated hunk from the index.

---

### Task 1: Harden Project Configuration And Authoring Defaults

**Files:**
- Modify: `.gitignore`
- Remove from Git tracking: `.hugo_build.lock`
- Modify: `config/_default/hugo.toml`
- Modify: `archetypes/default.md`

**Interfaces:**
- Consumes: Stack theme requirement `hugoVersion.min = "0.154.0"`.
- Produces: Chinese site language metadata, clean repository ignores, and a post archetype that requires stable URL and summary fields.

- [ ] **Step 1: Record the failing configuration checks**

Run:

```powershell
Select-String -Path config\_default\hugo.toml -Pattern 'languageCode = "zh-CN"'
Select-String -Path archetypes\default.md -Pattern '^slug:'
Select-String -Path archetypes\default.md -Pattern '^description:'
git check-ignore .hugo_build.lock tmp
```

Expected before implementation: the first three searches return no match;
`.hugo_build.lock` and `tmp` are not both ignored.

- [ ] **Step 2: Update repository ignores**

Change `.gitignore` to:

```gitignore
public
resources
.hugo_build.lock
tmp/
```

Run:

```powershell
git rm --cached -- .hugo_build.lock
```

Expected: the local lock file remains on disk and Git records only its removal
from tracking.

- [ ] **Step 3: Correct the site language and remove the sample Disqus service**

In `config/_default/hugo.toml`, replace:

```toml
languageCode = "en-us"
```

with:

```toml
languageCode = "zh-CN"
```

Delete this unused block:

```toml
[services.disqus]
    shortname = "hugo-theme-stack"
```

Keep `defaultContentLanguage = "zh"`, `hasCJKLanguage = true`, both permalink
rules, and all other site settings unchanged.

- [ ] **Step 4: Add stable metadata fields to the archetype**

Add these fields immediately after `title` in `archetypes/default.md`:

```yaml
slug: ""
description: ""
```

Do not remove or reorder the remaining article defaults.

- [ ] **Step 5: Verify the configuration task**

Run:

```powershell
Select-String -Path config\_default\hugo.toml -Pattern 'languageCode = "zh-CN"'
Select-String -Path config\_default\hugo.toml -Pattern 'services.disqus|shortname'
Select-String -Path archetypes\default.md -Pattern '^slug:|^description:'
git check-ignore -v .hugo_build.lock tmp
git diff --check
```

Expected: `zh-CN`, `slug`, and `description` are present; the Disqus search has
no result; both generated paths are ignored; `git diff --check` exits 0.

- [ ] **Step 6: Commit only project configuration changes**

Run:

```powershell
git add -- .gitignore config/_default/hugo.toml archetypes/default.md
git diff --cached --check
git diff --cached --name-only
git commit -m "chore: harden Hugo project defaults"
```

Expected staged paths: `.gitignore`, `.hugo_build.lock`,
`config/_default/hugo.toml`, and `archetypes/default.md`. No content file may
appear in this commit.

---

### Task 2: Migrate Existing Post Front Matter

**Files:**
- Modify: `content/post/lua-string.md`
- Modify: `content/post/aoi/index.md`
- Modify: `content/post/first-post/index.md`
- Modify: `content/post/flatbuffers/index.md`
- Modify: `content/post/lua-gc/index.md`
- Modify: `content/post/lua-gc-debt/index.md`
- Modify: `content/post/lua-gc-generation/index.md`
- Modify: `content/post/lua-struct/index.md`
- Modify: `content/post/lua-table/index.md`
- Modify: `content/post/protocol/index.md`
- Modify: `content/post/rpc/index.md`
- Modify: `content/post/sproto/index.md`

**Interfaces:**
- Consumes: the pre-migration permalink list captured by `hugo list all`.
- Produces: one explicit ASCII slug and one Chinese description per post, plus aliases for the five changed public URLs.

- [ ] **Step 1: Capture and preserve the old URL baseline**

Run:

```powershell
hugo list all | Tee-Object -Variable oldUrls
$oldUrls | Select-String '/p/'
```

Expected changed URL sources:

```text
/p/rpc-%E6%A1%86%E6%9E%B6/
/p/lua-%E5%A2%9E%E9%87%8Fgc-%E6%BC%94%E8%BF%9B%E4%BB%8E%E9%98%88%E5%80%BC%E6%A8%A1%E5%9E%8B%E5%88%B0-debt-%E5%80%BA%E5%8A%A1-%E6%A8%A1%E5%9E%8B/
/p/lua-%E5%88%86%E4%BB%A3-gc/
/p/lua-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E4%B8%8Egc%E5%AF%B9%E8%B1%A1/
/p/lua-%E5%A2%9E%E9%87%8F-gc/
```

- [ ] **Step 2: Confirm metadata checks fail before migration**

Run:

```powershell
rg --files-without-match '^slug: .+' content/post -g '*.md'
rg --files-without-match '^description: .+' content/post -g '*.md'
```

Expected: all twelve post files are reported because neither field currently
exists.

- [ ] **Step 3: Add exact slugs and descriptions**

Add the following values inside each post's existing front matter. Preserve
all fields and body content not listed here.

| File | Slug | Description |
|---|---|---|
| `content/post/lua-string.md` | `lua-string` | `从 TString 数据结构出发，梳理 Lua 短字符串与长字符串的存储、哈希和垃圾回收机制，以及字符串拼接与 Lua/C 交互中的常见开销。` |
| `content/post/aoi/index.md` | `aoi` | `从玩法和算法两个维度梳理 AOI 系统，比较九宫格、灯塔、十字链表与四叉树等方案，并说明九宫格 AOI 的数据结构和视野更新流程。` |
| `content/post/first-post/index.md` | `hello-world` | `记录建立这个博客的初衷，在技术快速发展的时代继续整理自己的生活、思考与成长。` |
| `content/post/flatbuffers/index.md` | `flatbuffers` | `介绍 FlatBuffers 的 vtable、内存布局、对齐与零拷贝原理，并说明它的兼容方式、性能特点和使用限制。` |
| `content/post/lua-gc/index.md` | `lua-gc` | `梳理 Lua 增量垃圾回收的三色标记、状态机、写屏障和双白机制，以及 pause 与 stepmul 对回收节奏的影响。` |
| `content/post/lua-gc-debt/index.md` | `lua-gc-debt` | `对比 Lua 5.1 的阈值模型与后续版本的 GC Debt 模型，说明步进计算、债务含义及其对增量回收节奏的影响。` |
| `content/post/lua-gc-generation/index.md` | `lua-gc-generation` | `梳理 Lua 分代 GC 的对象年龄、向后写屏障、Minor Collection、链表清扫，以及回退到 Major GC 的条件和优化思路。` |
| `content/post/lua-struct/index.md` | `lua-struct` | `从 TValue、GCObject、Closure、Proto 和 UpVal 等结构出发，梳理 Lua 基本类型与垃圾回收对象的内存组织。` |
| `content/post/lua-table/index.md` | `lua-table` | `结合 Lua Table 的数组与哈希结构，说明整数键归属、冲突处理、Rehash 过程，以及 pairs 和 ipairs 的迭代行为。` |
| `content/post/protocol/index.md` | `protocol-buffers` | `从 wire type、Varint 和 Length-delimited 编码出发，拆解 Protocol Buffers 的字段布局、序列化与反向解析过程。` |
| `content/post/rpc/index.md` | `rpc` | `从协议设计、序列化、网络传输、调用路由和异常治理五个层次，梳理 RPC 框架的组成与各层职责。` |
| `content/post/sproto/index.md` | `sproto` | `介绍 Sproto 的头部与数据区布局、内联编码和递归解析流程，并从性能、压缩率、GC 压力与生态几个方面对比 Protobuf。` |

Use double-quoted YAML strings:

```yaml
slug: "lua-gc"
description: "梳理 Lua 增量垃圾回收的三色标记、状态机、写屏障和双白机制，以及 pause 与 stepmul 对回收节奏的影响。"
```

- [ ] **Step 4: Add aliases only to posts whose URL changes**

Add these exact decoded paths to the matching front matter:

```yaml
# content/post/rpc/index.md
aliases:
    - "/p/rpc-框架/"

# content/post/lua-gc-debt/index.md
aliases:
    - "/p/lua-增量gc-演进从阈值模型到-debt-债务-模型/"

# content/post/lua-gc-generation/index.md
aliases:
    - "/p/lua-分代-gc/"

# content/post/lua-struct/index.md
aliases:
    - "/p/lua-数据类型与gc对象/"

# content/post/lua-gc/index.md
aliases:
    - "/p/lua-增量-gc/"
```

Do not add aliases to posts whose canonical URL is unchanged.

- [ ] **Step 5: Normalize Lua taxonomy spelling**

Change lowercase `lua` values to `Lua` in:

```text
content/post/lua-string.md
content/post/lua-gc-generation/index.md
content/post/lua-struct/index.md
```

Do not change any other category or tag.

- [ ] **Step 6: Verify source metadata and body preservation**

Run:

```powershell
rg --files-without-match '^slug: \"[a-z0-9-]+\"$' content/post -g '*.md'
rg --files-without-match '^description: \".+\"$' content/post -g '*.md'
rg -n '^\s+- "lua"$' content/post -g '*.md'
git diff --check -- content/post
git diff --word-diff=porcelain -- content/post
```

Expected: the first three commands produce no matches; `git diff --check`
exits 0. Review the word diff and confirm every new change outside pre-existing
user edits is inside front matter.

- [ ] **Step 7: Stage post metadata without staging user body edits**

Run:

```powershell
git add -- content/post
git diff --cached --word-diff=porcelain -- content/post
```

If the cached diff includes the pre-existing `lua-table` body edit, run:

```powershell
git restore --staged -- content/post/lua-table/index.md
git add -p -- content/post/lua-table/index.md
```

Accept only the front-matter hunk and reject the body hunk. Then run:

```powershell
git diff --cached --word-diff=porcelain -- content/post/lua-table/index.md
git diff --cached --check
```

Expected: cached post changes contain front matter only.

- [ ] **Step 8: Commit the metadata migration**

Run:

```powershell
git commit -m "content: stabilize post metadata and URLs"
```

Expected: the user's pre-existing body edit remains unstaged in the working
tree after the commit.

---

### Task 3: Document The Build Contract And Add CI

**Files:**
- Create: `README.md`
- Create: `.github/workflows/hugo-check.yml`

**Interfaces:**
- Consumes: Hugo baseline and source layout established by Tasks 1 and 2.
- Produces: fresh-clone setup documentation and a build-only CI gate.

- [ ] **Step 1: Confirm documentation and workflow are absent**

Run:

```powershell
Test-Path README.md
Test-Path .github\workflows\hugo-check.yml
```

Expected before implementation: both values are `False`.

- [ ] **Step 2: Create the root README**

Create `README.md` with this content:

````markdown
# 白山羊的博客

这是一个使用 Hugo 和 [Stack](https://github.com/CaiJimmy/hugo-theme-stack)
主题构建的个人博客，主要记录 Lua、游戏服务端和后端开发相关内容。站点由
Cloudflare Pages 根据 `main` 分支自动部署。

## 环境要求

- Hugo Extended 0.154.0 或更高兼容版本
- Git

主题通过 Git submodule 引入，首次克隆时需要递归初始化。

## 本地运行

```bash
git clone --recurse-submodules https://github.com/whitessheep/WBlog.git
cd WBlog
hugo server -D
```

如果仓库已经克隆，但主题目录为空，可以执行：

```bash
git submodule update --init --recursive
```

默认预览地址为 <http://localhost:1313/>。

## 生产构建

```bash
hugo --gc --minify --panicOnWarning
```

生成结果位于 `public/`，这个目录不进入版本控制。

## 项目结构

```text
content/              文章与独立页面
config/_default/      Hugo 和主题配置
assets/scss/          站点自定义样式
archetypes/           新文章模板
static/               直接复制的静态资源
themes/               Git submodule 管理的 Stack 主题
```

新文章需要填写稳定的英文 `slug` 和简短的中文 `description`。发布后不要随意
修改 `slug`；确实需要修改时，应使用 Hugo `aliases` 保留旧地址。

## Cloudflare Pages

Cloudflare Pages 继续使用现有 Git 集成部署，不由 GitHub Actions 发布。项目的
构建命令应设置为：

```text
hugo --gc --minify
```

同时在 Cloudflare Pages 环境变量中设置：

```text
HUGO_VERSION=0.154.0
```

仓库中的 GitHub Actions 只负责验证构建和主题兼容性，不需要 Cloudflare
Token。
````

- [ ] **Step 3: Create the validation-only GitHub Actions workflow**

Create `.github/workflows/hugo-check.yml`:

```yaml
name: Hugo build check

on:
  push:
    branches:
      - main
  pull_request:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Set up Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.154.0"
          extended: true

      - name: Build site
        run: hugo --gc --minify --panicOnWarning
```

The workflow must not contain `deploy`, `wrangler`, `pages`, `token`,
`secrets`, or artifact upload steps.

- [ ] **Step 4: Verify README and workflow consistency**

Run:

```powershell
rg -n '0\.154\.0|submodule update --init --recursive|hugo --gc --minify' README.md .github/workflows/hugo-check.yml
rg -ni 'deploy|wrangler|token|secrets|upload-artifact' .github/workflows/hugo-check.yml
git diff --check -- README.md .github/workflows/hugo-check.yml
```

Expected: version and build commands appear in both files; the forbidden CI
search produces no result; `git diff --check` exits 0.

- [ ] **Step 5: Commit documentation and CI**

Run:

```powershell
git add -- README.md .github/workflows/hugo-check.yml
git diff --cached --check
git diff --cached --name-only
git commit -m "ci: verify Hugo production builds"
```

Expected: only `README.md` and `.github/workflows/hugo-check.yml` are committed.

---

### Task 4: Verify Build, URLs, Aliases, And Working-Tree Safety

**Files:**
- Verify: all files changed by Tasks 1 through 3
- Verify generated output in a unique temporary directory

**Interfaces:**
- Consumes: completed configuration, post metadata, documentation, and CI.
- Produces: fresh evidence that the engineering baseline satisfies the design.

- [ ] **Step 1: Download a temporary compatible Hugo binary if needed**

If `hugo version` reports a version below `0.154.0`, run:

```powershell
$hugoVersion = "0.154.0"
$hugoDir = Join-Path $env:TEMP "hugo-$hugoVersion-extended"
$hugoZip = Join-Path $env:TEMP "hugo-$hugoVersion-extended.zip"
Invoke-WebRequest "https://github.com/gohugoio/hugo/releases/download/v$hugoVersion/hugo_extended_${hugoVersion}_windows-amd64.zip" -OutFile $hugoZip
Expand-Archive -LiteralPath $hugoZip -DestinationPath $hugoDir -Force
$hugo = Join-Path $hugoDir "hugo.exe"
& $hugo version
```

Expected: the command reports Hugo `v0.154.0` with `extended`.

- [ ] **Step 2: Run the strict production build**

Run with the compatible binary:

```powershell
$destination = Join-Path $env:TEMP ("myblog-verify-" + [guid]::NewGuid().ToString("N"))
& $hugo --gc --minify --panicOnWarning --destination $destination
```

Expected: exit code 0, no warnings, and a generated site in `$destination`.

- [ ] **Step 3: Verify canonical post URLs**

Run:

```powershell
& $hugo list all | Select-String '/p/'
```

Expected canonical post URLs:

```text
/p/lua-string/
/p/aoi/
/p/flatbuffers/
/p/sproto/
/p/protocol-buffers/
/p/rpc/
/p/lua-gc-debt/
/p/lua-gc-generation/
/p/lua-table/
/p/lua-struct/
/p/lua-gc/
/p/hello-world/
```

No canonical post URL may contain percent-encoded Chinese text.

- [ ] **Step 4: Verify generated alias redirects**

Check that these files exist under `$destination`:

```powershell
$aliases = [ordered]@{
  'p\rpc-框架\index.html' = 'https://weissgoat.pages.dev/p/rpc/'
  'p\lua-增量gc-演进从阈值模型到-debt-债务-模型\index.html' = 'https://weissgoat.pages.dev/p/lua-gc-debt/'
  'p\lua-分代-gc\index.html' = 'https://weissgoat.pages.dev/p/lua-gc-generation/'
  'p\lua-数据类型与gc对象\index.html' = 'https://weissgoat.pages.dev/p/lua-struct/'
  'p\lua-增量-gc\index.html' = 'https://weissgoat.pages.dev/p/lua-gc/'
}
$aliases.GetEnumerator() | ForEach-Object {
  $path = Join-Path $destination $_.Key
  if (-not (Test-Path -LiteralPath $path)) { throw "Missing alias: $($_.Key)" }
  $html = Get-Content -Raw -Encoding utf8 -LiteralPath $path
  $redirect = "http-equiv=refresh content=`"0; url=$($_.Value)`""
  if ($html -notmatch [regex]::Escape($redirect)) {
    throw "Wrong alias target: $($_.Key)"
  }
}
```

Expected: all five alias files exist and redirect to their exact absolute
canonical post URLs.

- [ ] **Step 5: Verify metadata completeness and repository safety**

Run:

```powershell
rg --files-without-match '^slug: \"[a-z0-9-]+\"$' content/post -g '*.md'
rg --files-without-match '^description: \".+\"$' content/post -g '*.md'
rg -n '^\s+- "lua"$' content/post -g '*.md'
git diff --check
git status --short
```

Expected: the first three commands produce no matches; `git diff --check`
exits 0; the user's pre-existing working-tree edits and untracked `index.txt`
remain visible unless the user handled them independently. `tmp/` no longer
appears because it is ignored.

- [ ] **Step 6: Review commit and content boundaries**

Run:

```powershell
git log -4 --oneline --stat
git diff HEAD~3..HEAD -- content/post
git status --short
```

Expected: project configuration, post front matter, and CI/documentation are
split into focused commits. The committed content diff changes front matter
only, and unrelated user body changes remain outside the implementation
commits.
