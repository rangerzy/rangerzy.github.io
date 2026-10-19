# Ranger 的技术手记

**x² + y² = z²**

基于 Hugo + LoveIt 的中文技术博客，发布地址：<https://rangerzy.github.io/>。
目前仅保留《数据备份详细设计》及其 9 张配图，支持搜索、标签、分类、评论、社交分享、明暗切换、文章目录、代码复制和数学公式。

## 本地预览

安装 **Hugo Extended 0.145.0**。主题固定为 **LoveIt v0.3.1**，请勿直接使用最新版 Hugo；LoveIt 0.3.X 的兼容范围是 Hugo 0.128.0–0.145.0。

```sh
git clone --recurse-submodules https://github.com/rangerzy/rangerzy.github.io.git
cd rangerzy.github.io
hugo server --environment production
```

已经克隆过仓库时，先初始化主题：

```sh
git submodule update --init --recursive
```

生成发布文件：

```sh
hugo --gc --minify --environment production
```

输出在 `public/`，该目录不提交到 Git。主题的 CSS、JavaScript、字体及中文搜索资源随网站发布，评论服务由 Giscus 提供。

## 首页外观

首页沿用 LoveIt 官方 Demo 的原生布局：圆形头像、打字机副标题、社交链接和文章横幅封面。顶部显示「Ranger 的技术手记」，头像下方显示 `x² + y² = z²`，中央不再重复站名。

- 头像：`assets/images/avatar.jpg`，来自自己的 GitHub 公开头像，随网站发布。
- 封面：`content/posts/data-backup-design/cover.jpg`，1600 × 480，通过文章的 `featuredImagePreview` 配置展示；正文原有 9 张配图完整保留。
- 封面使用内置 imagegen 生成。提示词：浅灰技术文档桌面拼贴，中央青绿色 `DATA BACKUP` 卡片，银色存储设备，柔和自然光，少量浅黄与珊瑚色纸签，适配 10∶3 横幅。使用 Hugo 图像处理生成发布用 JPEG。

## GitHub Pages 自动发布

1. 打开仓库 **Settings → Pages → Build and deployment**，将 **Source** 设为 **GitHub Actions**。
2. 将本次改造提交并推送到 `main`。
3. 在 **Actions → Build and deploy Hugo site** 查看结果；成功后访问 <https://rangerzy.github.io/>。

以后每次推送 `main` 都会触发 Hugo 编译和部署；上线需要等待 Actions 完成。对 `main` 的拉取请求仅验证构建，不部署。也可以从 Actions 手动触发，只有 `main` 的构建会发布。

工作流使用仓库自带的 `GITHUB_TOKEN`，无需个人访问令牌或部署密钥。部署失败时，先检查 Pages 发布源、Actions 权限及 `github-pages` 环境对 `main` 的部署限制。

## 启用 Giscus 评论

Giscus 使用当前公开仓库的 GitHub Discussions 保存评论，访客登录 GitHub 后发表评论。
完成以下一次性设置后评论才可用；未配置时不会加载无效的评论组件。

1. 在仓库 **Settings → General → Features** 启用 **Discussions**。
2. 安装 [Giscus GitHub App](https://github.com/apps/giscus)，仅授权 `rangerzy/rangerzy.github.io`。
3. 打开 <https://giscus.app/zh-CN>，输入 `rangerzy/rangerzy.github.io`，选择 **Announcements** 分类，页面映射选 **pathname**。
4. 将生成配置中的 `data-category-id` 填入 `hugo.toml` 的 `[params.page.comment.giscus]` → `categoryId`。
   仓库 ID 已根据 GitHub API 填好；可与生成的 `data-repo-id` 核对。
5. 提交并推送，触发重新部署。

也可以不修改配置文件，在仓库 **Settings → Secrets and variables → Actions → Variables** 添加 `GISCUS_CATEGORY_ID`。工作流仅在此变量有值时覆盖文件配置。本地预览可使用：

```sh
HUGO_PARAMS_PAGE_COMMENT_GISCUS_CATEGORYID='填写生成的 data-category-id' hugo server --environment production
```

这些 ID 是公开标识，不是密钥。安装 Giscus App 需要仓库所有者授权。

## 文章与静态资源

- 唯一文章：`content/posts/data-backup-design/index.md`，发布到 `/posts/data-backup-design/`。
- 文章配图与 Markdown 存放在同一目录，直接使用相对路径，如 `![说明](backup-01.png)`。
- 公共下载文件放到 `static/files/`。例如 `static/files/demo.pdf` 发布后可直接访问 `https://rangerzy.github.io/files/demo.pdf`。
- 原 mdBook 文章的 `.html` 地址保留跳转，指向迁移后的文章。
- 搜索使用本站生成的 `index.json` 和 Lunr 中文分词，无需搜索服务账号。

## 数学公式

已全站开启 KaTeX，并配置 Goldmark passthrough 保护公式中的下标、反斜杠等语法。行内公式：

```markdown
勾股定理：$x^2 + y^2 = z^2$。
```

独立公式块：

```markdown
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

也支持 `\(...\)` 和 `\[...\]`。单篇文章需要关闭公式渲染时，在 TOML front matter 中增加：

```toml
[math]
enable = false
```

副标题使用 Unicode 上标 `²`，无需等待公式脚本加载。原文章的 Mermaid 流程图与时序图继续由主题渲染。
