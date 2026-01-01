# 预制提示词模板库

本目录包含各种场景的预制提示词模板，用于生成高质量的网站配图。

## 模板分类

| 文件 | 场景 | 适用范围 |
|------|------|----------|
| `tech-style.md` | 科技风 | 技术博客、产品页面 |
| `illustration.md` | 扁平插画 | 公众号、知乎、小红书 |
| `hero-banner.md` | 网站 Hero | 首页大图、着陆页 |
| `blog-cover.md` | 博客封面 | 文章头图、专栏封面 |
| `icon-set.md` | 图标/徽标 | 功能图标、Logo |
| `diagram.md` | 流程图/架构图 | 技术文档、教程 |

## 使用方式

Claude 会根据用户需求自动选择合适的模板，也可以手动指定：

```
使用科技风模板生成一张配图
```

## 模板结构

每个模板包含：

```yaml
name: 模板名称
category: 分类
description: 适用场景描述

base_prompt: |
  基础提示词模板，包含占位符 {subject}, {style}, {color} 等

style_variants:
  - name: 变体名称
    modifier: 风格修饰词

color_palettes:
  - name: 配色方案名
    colors: ["#hex1", "#hex2", ...]

aspect_ratios:
  - 16:9  # 横版
  - 1:1   # 方形
  - 9:16  # 竖版
```
