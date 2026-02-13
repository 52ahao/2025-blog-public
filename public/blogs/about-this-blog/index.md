# 关于本博客项目：用前端 + GitHub 打造成的「完全属于你」的博客

这是一篇用这个博客系统写出来的、关于这个博客系统自己的文章。

## 这个项目是什么

本博客项目基于 [2025-blog-public](https://github.com/YYsuni/2025-blog-public) 的思路 fork 而来，核心特点可以概括为：

- **Next.js 16 + React 19** 的现代前端
- **内容全部存在你自己的 GitHub 仓库**里，没有第三方 CMS 或数据库
- **通过 GitHub App + 私钥**，在网页里就能完成写文、改配置、上传图片并自动推送到仓库
- 部署在 **Vercel** 或 **Cloudflare** 等平台，一次配置后，你拥有网站、仓库和所有内容

用原作者的话说：游戏资产不一定属于你，你只有使用权，但这个 blog **网站、内容、仓库一定是属于你的**。

---

## 技术栈一览

- **框架**：Next.js 16（App Router）、React 19
- **样式**：Tailwind CSS 4
- **动效**：Motion（原 Framer Motion）
- **状态**：Zustand
- **数据请求**：SWR
- **Markdown**：marked + Shiki 代码高亮、KaTeX 公式
- **部署**：支持 Vercel 与 OpenNext.js + Cloudflare

博客正文、站点配置、分享/项目/图片等数据，都以 **JSON / Markdown / 静态资源** 的形式放在仓库的 `public/`、`src/config/` 等目录下，通过 **GitHub API** 在前端完成「提交 → 创建 commit → 更新分支」的流程，从而无需自建后端或数据库。

---

## 内容是怎么「写进去」的：GitHub App 与私钥

要在前端改内容并生效，需要先让「网站」有权限往你的仓库里提交代码。做法是：

1. **创建一个 GitHub App**，只给当前博客仓库「写」权限，并生成一对密钥（Private Key）。
2. 在 **Vercel（或当前部署平台）** 配置环境变量：至少填 `NEXT_PUBLIC_GITHUB_OWNER` 和 `NEXT_PUBLIC_GITHUB_APP_ID`，若不用默认值再配 `REPO`、`BRANCH` 等。
3. 在网站里 **填入私钥**：多数页面右上角有「编辑」或配置入口，首次编辑时会要求输入 GitHub App 的 Private Key。私钥只用于在浏览器里签发 **JWT**，再换取 GitHub 的 **Installation Token**，用来调 GitHub API 推送内容，**不会**上传到你的仓库或第三方服务器（可配合项目里的「缓存私钥」选项，仅保存在本地 session 中）。

流程可以简化为：**私钥 → JWT → Installation Token → 调用 GitHub API 创建 blob / tree / commit → 更新分支**。推送成功后，Vercel 会触发重新部署，等部署完成刷新页面即可看到新内容。

---

## 你能在前端里「改」哪些东西

- **博客文章**：在 `/write` 写 Markdown，支持封面、标签、分类、摘要；可上传图片（先点 + 上传，再拖进正文），发布后写入 `public/blogs/{slug}/`（含 `index.md`、`config.json` 和图片），并更新 `public/blogs/index.json`。
- **站点配置**：首页不显眼的「配置」入口（或快捷键）里可改站点 meta、首页卡片开关、背景图、背景色、社交按钮、备案号等，对应 `src/config/site-content.json`、`card-styles.json` 以及 `public/` 下的 favicon、头像等。
- **首页布局**：支持拖拽卡片、保存偏移（仅本地/配置内生效）。
- **其他内容**：分享、项目、图片、片段、博主列表等，各有对应列表页和「创建/编辑」对话框，改完通过各自的 `push-*` 服务推送到仓库里的 JSON 或资源路径。

删除、批量删除博客等也会通过 GitHub API 删文件并更新 index，保证仓库和页面一致。

---

## 博客文章的数据结构

每篇文章在仓库里大致长这样：

- `public/blogs/{slug}/index.md`：正文 Markdown
- `public/blogs/{slug}/config.json`：标题、日期、标签、摘要、封面、分类、是否隐藏等
- `public/blogs/{slug}/*.png`（等）：文中或封面图片
- `public/blogs/index.json`：所有文章的索引（slug、title、tags、date、summary、cover、hidden、category 等），列表页和 RSS 都依赖它
- `public/blogs/categories.json`：分类列表（若启用分类）

阅读页通过 `loadBlog(slug)` 拉取 `config.json` + `index.md`，再用项目里的 Markdown 渲染管线（marked → 代码块/公式/图片等）输出页面；列表页和导航则读 `index.json`。

---

## 适合谁用

- 想有一个**自己完全可控**的博客，不依赖第三方内容平台或数据库。
- 能接受「用 GitHub 存内容、用 GitHub App 授权推送」这一套，并愿意在部署平台配置环境变量、在浏览器里配置一次私钥。
- 非前端也可以：按照 README 部署、创建 GitHub App、配好环境变量和私钥后，大部分操作都在网页里完成；只有少数定制（例如移除 Liquid Grass、改首页某个 Card 的文案）需要动仓库里的代码，README 里也有对应路径说明。

---

## 小结

这个博客项目把「内容在 GitHub、发布靠前端」做成了一整套可用的方案：技术栈现代、内容归属清晰、扩展点明确（新页面、新数据类型都可以仿照现有 `push-*` 和 JSON 结构加）。如果你正在找一种「前端 + GitHub 作为唯一数据源」的博客实现，不妨以这个项目为起点，按需删改和扩展。

*本文由本博客项目生成，用于介绍项目本身。*
