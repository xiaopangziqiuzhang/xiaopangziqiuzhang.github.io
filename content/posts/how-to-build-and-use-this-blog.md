+++
date = "2026-10-17"
title = "博客搭建与使用指南"
description = "本站的技术栈、搭建过程、仓库结构、写作与发布流程，一篇看懂。"
tags = ["Hugo", "GitHub Pages", "教程"]
categories = ["教程"]
series = ["博客使用"]
weight = 1
+++

这篇文章记录本站（[xiaopangziqiuzhang.github.io](https://xiaopangziqiuzhang.github.io/)）是如何从零搭建起来的，以及日常如何写文章、预览和发布。既是给自己的备忘，也希望能帮到想搭个人博客的你。

## 一、技术栈

| 组件 | 作用 |
|---|---|
| [Hugo](https://gohugo.io/) | 静态网站生成器，把 Markdown 编译成静态 HTML |
| [hugo-coder](https://github.com/luizdepra/hugo-coder) | 简洁的博客主题，支持亮/暗色、多语言、代码高亮 |
| GitHub 仓库 | 存放站点源码和文章 |
| GitHub Actions | 每次推送后自动构建网站 |
| GitHub Pages | 免费托管静态网站 |

整个流程是这样的：

```text
写 Markdown  →  git push  →  GitHub Actions 构建  →  GitHub Pages 发布
```

不需要买服务器，也不需要手动上传任何构建产物。

## 二、从零搭建的完整步骤

### 1. 安装 Hugo（必须是 extended 版）

hugo-coder 主题使用 SCSS 编译样式，所以需要 **Hugo extended** 版本。普通版会报错 `this feature is not available in your current Hugo version`。

macOS 上可以从 [Hugo Releases](https://github.com/gohugoio/hugo/releases) 下载 `hugo_extended_<版本>_darwin-universal.pkg`，也可以使用包管理器安装。本机是把官方 pkg 里的二进制解出来放到 `~/bin/hugo`，并将 `~/bin` 加进了 `PATH`：

```bash
hugo version
# hugo v0.168.0+extended darwin/arm64
```

### 2. 创建站点

```bash
hugo new site xiaopangziqiuzhang.github.io
cd xiaopangziqiuzhang.github.io
git init
```

### 3. 引入主题

```bash
git submodule add https://github.com/luizdepra/hugo-coder.git themes/hugo-coder
```

用 git submodule 而不是直接复制，好处是以后更新主题只需要一条命令，而且不会把主题的代码混进自己的提交历史。

### 4. 配置 `hugo.toml`

核心配置如下（完整内容见仓库里的 `hugo.toml`）：

```toml
baseURL = "https://xiaopangziqiuzhang.github.io/"
title = "xpz 的博客"
theme = "hugo-coder"
locale = "zh-cn"
defaultContentLanguage = "zh-cn"

[params]
author = "xpz"
description = "xpz 的个人博客"
dateFormat = "2006-01-02"
colorScheme = "auto"

[[menu.main]]
name = "关于"
url = "about/"
weight = 1

[[menu.main]]
name = "文章"
url = "posts/"
weight = 2
```

几个关键点：

- `baseURL` 必须写最终访问地址，否则 RSS、绝对链接会出错；
- `theme` 要和 `themes/` 下的目录名一致；
- `[[menu.main]]` 定义顶部导航菜单；
- `[[params.social]]` 定义页脚的社交图标链接。

### 5. 在 GitHub 创建仓库

仓库名有讲究：

- 命名为 **`<用户名>.github.io`**（例如 `xiaopangziqiuzhang.github.io`）→ 站点发布在 `https://<用户名>.github.io/` 根路径；
- 命名成其他名字 → 站点发布在 `https://<用户名>.github.io/<仓库名>/` 子路径，`baseURL` 要相应调整。

仓库需要设为 **Public**（免费账号的 Pages 只支持公开仓库）。

### 6. 添加自动部署工作流

创建 `.github/workflows/hugo.yml`，核心步骤是：

1. 安装指定版本的 Hugo extended；
2. `actions/checkout` 拉取代码（`submodules: recursive` 拉取主题）；
3. `hugo --gc --minify` 构建；
4. 上传构建产物；
5. 用 `actions/deploy-pages` 发布。

工作流需要的权限：

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

### 7. 开启 GitHub Pages

仓库 → **Settings** → **Pages** → **Build and deployment** → **Source** 选择 **GitHub Actions**。

> 注意：不要选 “Deploy from a branch”。否则 GitHub 会用 Jekyll 把仓库源码原样构建一遍，得到一个没有首页的站点（访问根路径 404）。

### 8. 推送上线

```bash
git add -A
git commit -m "初始化博客"
git branch -M main
git remote add origin git@github.com:<用户名>/<用户名>.github.io.git
git push -u origin main
```

如果使用 SSH 推送，需要先把公钥加到 GitHub（Settings → SSH and GPG keys），再用下面的命令验证：

```bash
ssh -T git@github.com
# Hi <用户名>! You've successfully authenticated...
```

推送后，GitHub Actions 会自动构建并发布，1～2 分钟后访问 `https://<用户名>.github.io/` 即可。

## 三、仓库结构

```text
xiaopangziqiuzhang.github.io/
├── .github/
│   └── workflows/
│       └── hugo.yml        # 自动构建 + 部署工作流
├── content/                # 所有内容（Markdown）
│   ├── posts/              # 博客文章
│   └── about.md            # 关于页
├── themes/
│   └── hugo-coder/         # 主题（git submodule）
├── hugo.toml               # 站点配置
├── .gitignore              # 忽略 public/ 等构建产物
└── .gitmodules             # submodule 配置
```

`public/` 是本地构建产物，已经在 `.gitignore` 中忽略，**不需要提交**——线上版本由 GitHub Actions 构建。

## 四、如何写文章

### 1. 创建文章

```bash
cd ~/wangyongshuai/github/xpz.github.io
hugo new content posts/my-new-post.md
```

会生成类似下面的文件：

```toml
+++
draft = true
date = 2026-10-17T11:42:53+08:00
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++
```

字段说明：

| 字段 | 说明 |
|---|---|
| `title` | 文章标题 |
| `date` | 发布日期，决定文章排序 |
| `description` | 摘要，用于列表页和 SEO |
| `tags` / `categories` | 标签和分类，自动生成归档页 |
| `slug` | 自定义 URL 路径（留空则用文件名） |
| `externalLink` | 填了之后列表里点击直接跳转到外部链接 |
| `series` | 系列文章分组 |
| `draft` | `true` 表示草稿，**不会出现在线上站点** |

### 2. 草稿与发布

新文章默认 `draft = true`。确认要发布时，改成 `false` 或删掉这一行。

- 本地预览：`hugo server -D`（带 `-D` 才会显示草稿）
- 线上构建：工作流中没有 `-D`，所以草稿不会被发布

### 3. Markdown 常用语法

````markdown
# 一级标题（正文里一般从二级开始用）
## 二级标题

**粗体**、*斜体*、`行内代码`

- 无序列表
- 第二项

1. 有序列表
2. 第二项

[链接文字](https://example.com)

![图片说明](/images/photo.png)

> 引用文字

```python
print("代码块支持语法高亮")
```
````

图片放到 `static/images/` 目录下，引用时写 `/images/文件名`。

表格：

```markdown
| 列一 | 列二 |
|---|---|
| 内容 | 内容 |
```

### 4. 本地预览

```bash
cd ~/wangyongshuai/github/xpz.github.io
hugo server -D
```

浏览器打开 <http://localhost:1313/>，修改文件保存后页面会自动刷新。

## 五、发布文章的完整流程

```bash
# 1. 写文章
hugo new content posts/my-new-post.md
# 编辑 content/posts/my-new-post.md，把 draft 改为 false

# 2. 本地预览确认
hugo server -D

# 3. 提交并推送
git add -A
git commit -m "新增文章：我的新文章"
git push

# 4. 等待 1~2 分钟，GitHub Actions 自动构建发布
```

可以在仓库的 **Actions** 页面查看构建进度，绿色 ✅ 表示发布成功。

## 六、常见问题

| 问题 | 排查方法 |
|---|---|
| 线上访问 404 | 检查 Settings → Pages → Source 是否为 **GitHub Actions** |
| 构建失败 | 打开 Actions → 失败的 run，查看红色步骤的日志 |
| 本地提示找不到 `hugo` | 新开一个终端，或执行 `source ~/.zshrc`；用 `which hugo` 确认 |
| 改了文章但线上没变 | 确认已 `git push`；到 Actions 页面看是否有新的运行 |
| 样式/内容没有更新 | 强制刷新浏览器（`Cmd + Shift + R`） |
| 主题报 SCSS 编译错误 | 确认使用的是 Hugo **extended** 版本 |
| 文章列表没显示新文章 | 检查 front matter 里 `draft` 是否为 `true`、`date` 是否写错 |

### 更新主题

```bash
git submodule update --remote themes/hugo-coder
git add themes/hugo-coder
git commit -m "更新 hugo-coder 主题"
git push
```

### 修改站点信息

站点标题、作者、描述、菜单、社交链接都在 `hugo.toml` 中修改，提交推送后自动生效。

## 七、参考资料

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [hugo-coder 主题](https://github.com/luizdepra/hugo-coder)
- [GitHub Pages 文档](https://docs.github.com/pages)
- [GitHub Actions 文档](https://docs.github.com/actions)
- [Markdown 基本语法](https://www.markdownguide.org/basic-syntax/)
