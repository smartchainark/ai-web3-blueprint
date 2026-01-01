# 扁平插画模板 (Flat Illustration)

适用于公众号、知乎、小红书等内容平台配图。

## 基础提示词

```yaml
name: flat-illustration
category: 插画/设计
description: 扁平化设计风格，亲和力强，适合大众阅读

base_prompt: |
  A flat design illustration of {subject},
  featuring simple geometric shapes, soft rounded edges,
  and a warm, inviting color palette.
  Style: {style}, friendly, approachable.
  Characters (if any): minimal facial features, expressive poses.
  Background: {background_style}.
  Quality: vector-style, clean edges, high resolution.
```

## 风格变体

### 1. 温暖治愈 (Warm & Cozy)
```
暖色调，圆润线条，阳光感
适合生活方式、个人成长类内容
```

**修饰词**: `warm tones, soft shadows, cozy atmosphere, golden hour lighting`

### 2. 商务专业 (Business Pro)
```
蓝灰色调，简洁现代，专业感
适合职场、商业类内容
```

**修饰词**: `corporate blue, professional setting, clean lines, business casual`

### 3. 活力创意 (Vibrant Creative)
```
高饱和度，对比色，充满活力
适合创意、设计、年轻受众
```

**修饰词**: `vibrant colors, bold contrast, creative energy, dynamic composition`

### 4. 自然清新 (Nature Fresh)
```
绿色植物元素，自然光感
适合健康、环保、休闲类内容
```

**修饰词**: `nature elements, green plants, fresh air feeling, organic shapes`

## 人物风格

### 极简人物
```
无面部细节，用颜色和姿态表达情感
适合普适性强的场景
```

### 卡通风格
```
大眼睛，夸张表情，可爱比例
适合轻松有趣的内容
```

### 写实插画
```
相对真实的人物比例，细节更多
适合专业、正式的内容
```

## 配色方案

### 暖阳橙 (Sunny Orange)
```yaml
colors:
  primary: "#ff6b35"
  secondary: "#f7c59f"
  accent: "#efa94a"
  background: "#fff5eb"
  shadow: "#e8d5c4"
```

### 薄荷绿 (Mint Green)
```yaml
colors:
  primary: "#2ec4b6"
  secondary: "#8ecae6"
  accent: "#95d5b2"
  background: "#f0fff4"
  shadow: "#d8e2dc"
```

### 梦幻紫 (Dream Purple)
```yaml
colors:
  primary: "#9b5de5"
  secondary: "#f15bb5"
  accent: "#fee440"
  background: "#faf5ff"
  shadow: "#e9d5ff"
```

### 天空蓝 (Sky Blue)
```yaml
colors:
  primary: "#219ebc"
  secondary: "#8ecae6"
  accent: "#ffb703"
  background: "#f0f9ff"
  shadow: "#bae6fd"
```

## 常用场景

### 工作场景
```
person working at desk, laptop, coffee cup,
home office, coworking space, productive atmosphere
```

### 学习成长
```
reading book, taking notes, lightbulb moment,
graduation, skill upgrade, knowledge acquisition
```

### 团队协作
```
people in meeting, brainstorming, teamwork,
puzzle pieces fitting, high five, collaboration
```

### 生活休闲
```
relaxing at home, outdoor activities, hobbies,
meditation, self-care, work-life balance
```

## 示例提示词

### 示例 1: 远程办公
```
A flat design illustration of a person working from home,
sitting at a cozy desk with a laptop, coffee cup, and plant,
warm orange and cream color palette,
simple geometric shapes, soft rounded edges,
window showing sunny day outside,
Style: warm & cozy, friendly, approachable.
Background: simple room interior with minimal details.
Quality: vector-style, clean edges, 16:9 aspect ratio.
```

### 示例 2: 团队协作
```
A flat design illustration of a diverse team collaborating,
four people gathered around a table with documents and devices,
each person in different pose showing engagement,
corporate blue and fresh green color palette,
Style: business pro, professional, inclusive.
Background: modern office with glass windows.
Quality: vector-style, clean lines, 1:1 aspect ratio.
```

## 尺寸建议

| 平台 | 比例 | 像素 |
|------|------|------|
| 公众号封面 | 2.35:1 | 900x383 |
| 公众号文中 | 16:9 | 1080x608 |
| 小红书 | 3:4 | 1080x1440 |
| 知乎回答 | 16:9 | 1200x675 |
