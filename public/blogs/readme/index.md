# 本博客部署与使用指南（README）

本项目从 [YYsuni/2025-blog-public](https://github.com/YYsuni/2025-blog-public) fork 而来，部署指南参考：[yysuni 的 readme](https://www.yysuni.com/blogs/readme)。

本项目使用 **GitHub App** 管理仓库内容，请妥善保管后续创建的 **Private Key**，不要上传到公开网络。

---

## 0. Fork 项目

请先 Fork 本仓库到你自己的 GitHub 账号下：

- 本仓库地址：**https://github.com/52ahao/2025-blog-public**
- 在 GitHub 页面点击 **Fork**，即可得到 `你的用户名/2025-blog-public`（或你改名的仓库）。后续可随时从上游同步更新。

---

## 1. 安装

可以不做本地开发，直接部署后再配置环境变量。环境变量名对应如下（具体见 `src/consts.ts`）：

```ts
export const GITHUB_CONFIG = {
  OWNER: process.env.NEXT_PUBLIC_GITHUB_OWNER || '',
  REPO: process.env.NEXT_PUBLIC_GITHUB_REPO || '',
  BRANCH: process.env.NEXT_PUBLIC_GITHUB_BRANCH || 'main',
  APP_ID: process.env.NEXT_PUBLIC_GITHUB_APP_ID || '-',
} as const
```

若需本地开发，执行：

```bash
pnpm i
```

---

## 2. 部署

支持 **Vercel** 与 **Cloudflare (Pages/Workers)** 两种方式，任选其一即可。

### 2.1 部署到 Vercel

1. 打开 [Vercel](https://vercel.com)，登录后点击 **Add New… → Project**。
2. **Import** 你 Fork 后的 GitHub 仓库（如 `你的用户名/2025-blog-public`）。
3. 无需改构建配置，直接点击 **Deploy**。约 1 分钟后会得到预览域名，例如：`https://xxx.vercel.app`。
4. 部署完成后，到 **Settings → Environment Variables** 添加环境变量（见下方「环境变量」）。保存后建议 **Redeploy** 一次使变量生效。

之后每次推送到 GitHub，Vercel 会自动重新部署。

### 2.2 部署到 Cloudflare（OpenNext + Wrangler）

本项目已集成 [@opennextjs/cloudflare](https://opennext.js.org/cloudflare)，可将 Next.js 构建为运行在 Cloudflare Workers/Pages 上的应用。

#### 方式 A：Cloudflare Pages（推荐，与 Git 联动）

1. 打开 [Cloudflare Dashboard](https://dash.cloudflare.com) → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**。
2. 选择你 Fork 的仓库（如 `你的用户名/2025-blog-public`）。
3. 配置构建设置：
   - **Framework preset**：None
   - **Build command**：`pnpm run build:cf`
   - **Build output directory**：`.open-next`
   - **Root directory**：留空（或 `/`）
4. 在 **Environment variables** 中添加 `NEXT_PUBLIC_GITHUB_OWNER`、`NEXT_PUBLIC_GITHUB_APP_ID` 等（见下方「环境变量」）。
5. 保存并部署。注意：Cloudflare Pages 默认可能期望静态输出。若使用 **OpenNext Cloudflare** 的 Worker 模式，需在仓库中配置 **Build configuration** 使用 `wrangler pages deploy` 或选用「Wrangler」构建（见 [OpenNext Cloudflare 文档](https://opennext.js.org/cloudflare)）。

#### 方式 B：本地构建 + Wrangler 部署

1. 克隆你的 Fork 并安装依赖：
   ```bash
   git clone https://github.com/你的用户名/2025-blog-public.git
   cd 2025-blog-public
   pnpm i
   ```
2. 在项目根目录创建或编辑 `wrangler.toml`（本项目已包含），确认 `name` 等与你在 Cloudflare 上创建的应用一致。
3. 配置环境变量（二选一）：
   - 在 **Cloudflare Dashboard** 中为该 Worker/Pages 项目设置 **Variables**（如 `NEXT_PUBLIC_GITHUB_OWNER`、`NEXT_PUBLIC_GITHUB_APP_ID`）；或
   - 在本地 `.dev.vars`（不提交到 Git）中写：
     ```
     NEXT_PUBLIC_GITHUB_OWNER=你的GitHub用户名
     NEXT_PUBLIC_GITHUB_APP_ID=你的GitHub_App_ID
     ```
4. 构建并部署：
   ```bash
   pnpm run build:cf   # 使用 OpenNext 构建出 .open-next
   pnpm run deploy     # 或: npx wrangler deploy
   ```
5. 若使用 **Cloudflare Pages** 且通过 Git 接入，可在 Dashboard 中将 **Build command** 设为 `pnpm run build:cf`，**Build output directory** 设为 `.open-next`，并确保 Pages 使用兼容 Node/OpenNext 的运行时（按官方文档配置）。

本地预览：

```bash
pnpm run preview
```

---

### 环境变量（Vercel / Cloudflare 通用）

| 变量名 | 说明 | 必填 |
|--------|------|------|
| `NEXT_PUBLIC_GITHUB_OWNER` | GitHub 用户名或组织 | 是 |
| `NEXT_PUBLIC_GITHUB_APP_ID` | GitHub App 的 App ID | 是 |
| `NEXT_PUBLIC_GITHUB_REPO` | 仓库名，默认与项目同名 | 否 |
| `NEXT_PUBLIC_GITHUB_BRANCH` | 分支，默认 `main` | 否 |

不会设置环境变量时，可直接改仓库里的 `src/consts.ts`（因仓库公开，仅适合个人使用）。

---

## 3. 创建 GitHub App 并关联仓库

1. GitHub 右上角头像 → **Settings** → 左侧最下方 **Developer settings** → **GitHub Apps** → **New GitHub App**。
2. 填写：
   - **GitHub App name** / **Homepage URL**：随意，可不填。
   - **Webhook**：可关闭。
   - **Repository permissions**：为 **Contents** 勾选 **Read and write**。
3. 创建后，在 **Private keys** 中 **Generate a private key**，下载并妥善保存；记下页面上的 **App ID**。
4. 进入 **Install App**，只选择**当前博客仓库**（你 Fork 的那一个），完成安装。

详细步骤与截图可参考：[yysuni 的 readme - 创建 Github App](https://www.yysuni.com/blogs/readme)。

---

## 4. 完成

部署并配置好环境变量与 GitHub App 后，在网站中通过「编辑」入口填入 **Private Key**，即可在前端直接改内容（文章、配置、分享等）。内容会通过 GitHub API 提交到你的仓库。

**提示**：前端保存成功后，需等 Vercel/Cloudflare 重新部署完成，再刷新页面才能看到最新内容。

---

## 5. 删除示例内容

使用本项目后，建议先在前端删除自带的示例 blog / 分享等，或通过「批量删除」清理。

---

## 6. 配置

- 大部分页面右上角有「编辑」按钮，需先配置 **Private Key** 才能提交。
- 首页有不显眼的「配置」入口，可配置站点信息、首页卡片、背景、社交按钮、备案号等。

---

## 7. 写 Blog

- 在 **写文章** 页面用 Markdown 撰写，支持封面、标签、分类、摘要。
- 图片建议：先点击 **+** 上传（推荐宽度 ≤1200px、先压缩），再将上传好的图片拖入正文。发布前可点右上角预览查看效果。

---

## 8. 写给非前端

以下为通过改代码完成的配置，适合愿意动仓库的用户。

### 8.1 移除 Liquid Grass

在 `src/layout/index.tsx` 中删除与 `LiquidGrass` 相关的两行（动态 import 与组件使用）。

### 8.2 配置首页内容

首页结构在 `src/app/(home)/`，入口为 `src/app/(home)/page.tsx`。各块对应不同 Card 组件（如 `hi-card.tsx`），需要改哪块就编辑对应文件。

### 8.3 导航 Card

全局导航 Card 在 `src/components/nav-card.tsx`。

---

## 9. 参考与致谢

- 部署与使用思路参考：[**yysuni 的 readme**](https://www.yysuni.com/blogs/readme)
- 上游项目：[YYsuni/2025-blog-public](https://github.com/YYsuni/2025-blog-public)

游戏资产不一定属于你，你只有**使用权**；但这个 blog 的**网站、内容与仓库**都可以完全属于你。
