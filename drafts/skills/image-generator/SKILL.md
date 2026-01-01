---
name: image-generator
description: AI图片生成器，支持即梦和Banana等多种生图引擎。当用户说"生成图片"、"配图"、"generate image"、"AI画图"时触发此技能。可异步生成并自动插入到文章中。
---

# 图片生成器 (Image Generator)

支持多种 AI 生图引擎，为文章配图提供一站式解决方案。

## 支持的生图引擎

### 即梦 (Jimeng)

```yaml
engine: jimeng
api_endpoint: https://jimeng.jianying.com/ai-tool/image
features:
  - 中文提示词优化
  - 多种艺术风格
  - 快速生成（~10秒）
pricing: 按次计费
```

**调用示例**：
```
使用即梦生成：现代化办公室场景，程序员在使用AI工具，科技蓝配色
```

### Banana AI

```yaml
engine: banana
api_endpoint: https://api.banana.dev
features:
  - Stable Diffusion 模型
  - 高度可定制
  - 批量生成
pricing: 按秒计费
```

**调用示例**：
```
使用 Banana 生成：digital illustration of automation workflow, tech blue theme, 4K
```

## 使用方式

### 1. 单图生成

```
生成图片：一个创作者从繁琐工作中解放，阳光洒进房间
```

### 2. 批量生成（异步）

```
为这篇文章的 3 个金句生成配图
```

### 3. 结合文章生成器

```
生成技术型文章并配图
```

## 图片提示词结构

```yaml
prompt:
  subject: 主体描述
  style: 艺术风格
  color_palette: 色彩方案
  mood: 情绪氛围
  technical:
    aspect_ratio: 比例
    quality: 质量要求
```

## 支持的输出格式

| 用途 | 比例 | 尺寸 |
|------|------|------|
| 文章头图 | 16:9 | 1920x1080 |
| 社交卡片 | 1:1 | 1080x1080 |
| 小红书 | 3:4 | 1080x1440 |
| 公众号封面 | 2.35:1 | 900x383 |

## 异步工作流

```
1. 提交生图任务 → 获取 Task ID
2. 继续处理其他任务（非阻塞）
3. 轮询检查状态
4. 图片就绪后自动替换占位符
```

## 集成能力

- 与 `article-generator` 联动，自动为金句配图
- 生成的图片可直接用于发布流程
- 支持 CDN 上传和 URL 返回

## 前置要求

- 即梦账号 或 Banana API Key
- 网络连接
- 可选：CDN 上传配置
