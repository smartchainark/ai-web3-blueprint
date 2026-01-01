---
name: article-generator
description: AI驱动的多风格文章生成器。当用户说"生成文章"、"转换风格"、"generate article"、"多风格改写"时触发此技能。支持技术型、故事型、产品型三种风格并行生成，自动提取金句并生成图片提示词。
---

# 文章生成器 (Article Generator)

将源文档自动转换为多种风格的文案，配合 AI 生图和模板定制。

## 核心能力

### 1. 三种风格转换

| 风格 | 触发词 | 输出特点 |
|------|--------|----------|
| 技术型 | "技术文" / "technical" | 代码示例、数据对比、精确术语 |
| 故事型 | "故事" / "storytelling" | 场景叙事、情感共鸣、转折起伏 |
| 产品型 | "产品文" / "product" | 卖点突出、价值驱动、CTA明确 |

### 2. 并行生成

```javascript
// 并行启动 3 个 Agent
const styles = ['technical', 'storytelling', 'product'];
const tasks = styles.map(style => generateStyle(sourceDoc, style));
await Promise.all(tasks);
```

### 3. 金句提取

自动识别并标记金句：
- 技术型：`> 💡 **技术洞察**：[内容]`
- 故事型：`> ✨ **灵感时刻**：[内容]`
- 产品型：`> 🚀 **核心价值**：[内容]`

### 4. 图片提示词生成

为每个金句生成配图提示词，包含：
- 主体描述
- 风格定义
- 色彩方案
- 技术参数

## 使用示例

```
# 基础用法
将 docs/article.md 生成 3 种风格

# 指定风格
将这篇文章改写成故事型

# 完整流水线
生成 3 种风格并配图，准备发布到知乎
```

## 输出结构

```
drafts/generated-articles/
├── technical-style.md    # 技术型文案
├── storytelling-style.md # 故事型文案
├── product-style.md      # 产品型文案
└── quotes-and-prompts.md # 金句与图片提示词
```

## 集成能力

- 与 `image-generator` Skill 联动生图
- 与 `zhihu-publisher` / `juejin-publisher` 联动发布
- 支持二次加工模板定制

## 前置要求

- Claude Code 环境
- 可选：MCP Playwright（用于自动发布）
- 可选：即梦/Banana API（用于生图）
