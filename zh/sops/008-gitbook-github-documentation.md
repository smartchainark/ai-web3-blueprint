# GitBook + GitHub 文档发布工作流

> 使用 GitBook 结构 + GitHub 托管，实现双语技术文档的标准化管理与发布。

**文档编号**: 008
**日期**: 2025-12-29
**标签**: `GitBook` `GitHub` `文档` `双语` `HonKit`

---

## 概述

本 SOP 介绍如何将 GitBook 目录结构与 GitHub 仓库结合，实现：

- 双语文档的标准化管理
- 本地预览与在线发布
- 版本控制与协作

## 目录结构

```
project/
├── book.json           # GitBook 配置
├── SUMMARY.md          # 根目录导航（多语言入口）
├── README.md           # 英文首页 + GitHub 展示
├── README.zh.md        # 中文首页
├── en/                 # 英文文档
│   ├── README.md       # 英文概览
│   ├── SUMMARY.md      # 英文导航
│   ├── sops/           # SOP 文档
│   ├── thoughts/       # 思考随笔
│   └── resources/      # 资源（术语表、工具）
├── zh/                 # 中文文档
│   ├── README.md       # 中文概览
│   ├── SUMMARY.md      # 中文导航
│   ├── sops/
│   ├── thoughts/
│   └── resources/
└── .gitignore          # 忽略 _book/ 等构建产物
```

## 核心文件

### 1. book.json

GitBook/HonKit 的配置文件：

```json
{
  "title": "项目名称",
  "description": "项目描述",
  "author": "作者",
  "language": "en",
  "structure": {
    "readme": "README.md",
    "summary": "SUMMARY.md"
  },
  "links": {
    "sidebar": {
      "GitHub": "https://github.com/your/repo"
    }
  },
  "plugins": ["-sharing"]
}
```

**注意**：
- `plugins: ["-sharing"]` 禁用分享按钮，避免渲染问题
- 复杂插件可能导致 HonKit 报错，建议保持简洁

### 2. SUMMARY.md（根目录）

多语言入口导航：

```markdown
# Summary

* [Introduction](README.md)

## English

* [Overview](en/README.md)
* [SOPs & Guides](en/sops/001-first-doc.md)
  * [First Doc](en/sops/001-first-doc.md)
  * [Second Doc](en/sops/002-second-doc.md)
* [Thoughts](en/thoughts/README.md)
* [Glossary](en/resources/glossary.md)

## 中文

* [概览](zh/README.md)
* [SOP 与指南](zh/sops/001-first-doc.md)
  * [第一篇](zh/sops/001-first-doc.md)
  * [第二篇](zh/sops/002-second-doc.md)
* [思考随笔](zh/thoughts/README.md)
* [术语表](zh/resources/glossary.md)
```

### 3. README.md（GitHub 首页）

同时作为 GitHub 仓库首页和 GitBook 入口：

```markdown
# Project Name

[![License](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)]()
![GitHub stars](https://img.shields.io/github/stars/user/repo?style=social)

> 项目描述

[中文版](./README.zh.md)

---

## 文档索引

| # | Title | Description | Date |
|---|-------|-------------|------|
| 001 | [Doc Title](./en/sops/001-doc.md) | Description | 2025-12-29 |

---

**Author**: name
**GitHub**: https://github.com/user/repo
```

## 操作流程

### 步骤 1：初始化项目

```bash
# 创建目录结构
mkdir -p en/sops en/thoughts en/resources
mkdir -p zh/sops zh/thoughts zh/resources

# 创建核心文件
touch book.json SUMMARY.md README.md README.zh.md
touch en/README.md en/SUMMARY.md
touch zh/README.md zh/SUMMARY.md
```

### 步骤 2：安装 HonKit

由于 `gitbook-cli` 不兼容 Node.js 18+，使用 HonKit 替代：

```bash
# 全局安装
npm install -g honkit

# 或项目本地安装
npm install honkit --save-dev
```

### 步骤 3：本地预览

```bash
# 启动开发服务器
honkit serve

# 访问 http://localhost:4000
```

### 步骤 4：构建静态文件

```bash
# 构建到 _book/ 目录
honkit build

# 输出目录结构
ls _book/
```

### 步骤 5：Git 版本控制

```bash
# .gitignore 添加构建产物
echo "_book/" >> .gitignore
echo "node_modules/" >> .gitignore

# 提交代码
git add .
git commit -m "docs: add new documentation"
git push
```

## 文档命名规范

### 文件名

```
{编号}-{英文标题小写连字符}.md

示例：
001-notion-mcp-setup.md
002-internal-comms-to-exports.md
008-gitbook-github-documentation.md
```

### 文档结构

```markdown
# 标题

> 一句话描述

**文档编号**: NNN
**日期**: YYYY-MM-DD
**标签**: `标签1` `标签2`

---

## 概述

...

## 步骤 1：XXX

...

## 常见问题

...

## 参考资源

- [链接](url)
```

## 发布选项

### 选项 1：GitHub Pages

```bash
# 安装 gh-pages 插件
npm install gh-pages --save-dev

# 部署到 GitHub Pages
npx gh-pages -d _book
```

### 选项 2：GitBook.com

1. 在 [gitbook.com](https://www.gitbook.com) 创建 Space
2. 连接 GitHub 仓库
3. 自动同步发布

### 选项 3：静态托管

将 `_book/` 目录部署到任意静态托管服务：
- Vercel
- Netlify
- Cloudflare Pages

## 常见问题

### Q: HonKit 报 Maximum call stack size exceeded

**A**: 通常是插件问题，简化 `book.json`：

```json
{
  "plugins": ["-sharing"]
}
```

### Q: gitbook-cli 安装失败

**A**: `gitbook-cli` 不兼容 Node.js 18+，使用 HonKit 替代：

```bash
npm uninstall -g gitbook-cli
npm install -g honkit
```

### Q: 中文文件名在 Telegram 显示为链接

**A**: Telegram 会将 `.md` 结尾的文本识别为链接，通知时去掉扩展名：

```
文件：zh/sops/008-gitbook-github-documentation
```

## 参考资源

- [HonKit 官方文档](https://github.com/honkit/honkit)
- [GitBook 遗留文档](https://github.com/GitbookIO/gitbook)
- [GitHub Pages 文档](https://docs.github.com/en/pages)
