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
