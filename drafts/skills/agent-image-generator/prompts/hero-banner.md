# 网站 Hero Banner 模板

适用于网站首页大图、着陆页、产品展示页。

## 基础提示词

```yaml
name: hero-banner
category: 网站/着陆页
description: 大尺寸横幅图，视觉冲击力强，适合网站首屏

base_prompt: |
  A stunning hero banner image for {subject},
  wide composition (21:9 or 16:9 aspect ratio),
  {style} aesthetic with {mood} atmosphere.
  Key visual element: {focal_point} positioned using rule of thirds.
  Background: {background_description}.
  Color theme: {color_theme}.
  Lighting: {lighting_style}.
  Space for text overlay on {text_area_position}.
  Quality: ultra high resolution, 4K, professional photography style.
```

## 风格变体

### 1. 科技产品 (Tech Product)
```
深色背景，产品居中或黄金分割点
光效突出产品轮廓，未来感十足
```

**修饰词**: `product showcase, dark gradient, rim lighting, sleek modern`

### 2. SaaS 服务 (SaaS Service)
```
渐变背景，抽象几何元素
留白区域放置文案，专业大气
```

**修饰词**: `gradient background, abstract shapes, ample text space, corporate`

### 3. 创意工作室 (Creative Studio)
```
大胆配色，艺术感强
视觉张力，吸引眼球
```

**修饰词**: `bold colors, artistic, creative energy, eye-catching`

### 4. 自然生态 (Nature & Eco)
```
自然场景，绿色生态元素
清新舒适，环保主题
```

**修饰词**: `nature scene, green elements, fresh air, sustainability`

## 构图原则

### 留白区域
```
Hero Banner 需要预留文字区域：
- 左侧留白：适合从左到右阅读的文案
- 中央留白：适合居中大标题
- 渐变遮罩：图片上加深色渐变，确保文字可读
```

### 视觉焦点
```
使用三分法或黄金分割：
- 主体不要正中，稍微偏移更有动感
- 眼睛/关键元素放在交叉点
- 留出呼吸空间
```

## 配色方案

### 深邃蓝 (Deep Blue)
```yaml
colors:
  gradient_start: "#0f172a"
  gradient_end: "#1e3a5f"
  accent: "#3b82f6"
  text: "#ffffff"
  cta_button: "#22d3ee"
```

### 日落渐变 (Sunset Gradient)
```yaml
colors:
  gradient_start: "#7c3aed"
  gradient_end: "#ec4899"
  accent: "#fbbf24"
  text: "#ffffff"
  cta_button: "#f97316"
```

### 极简白 (Minimal White)
```yaml
colors:
  gradient_start: "#ffffff"
  gradient_end: "#f1f5f9"
  accent: "#0f172a"
  text: "#1e293b"
  cta_button: "#3b82f6"
```

### 暗夜金 (Dark Gold)
```yaml
colors:
  gradient_start: "#0c0a09"
  gradient_end: "#1c1917"
  accent: "#fbbf24"
  text: "#fef3c7"
  cta_button: "#f59e0b"
```

## 常用主题

### 产品发布
```
new product reveal, spotlight effect, premium feel,
elegant presentation, launch announcement
```

### AI/技术服务
```
AI assistant interface, neural network visualization,
futuristic tech, innovation, smart automation
```

### 团队/关于我们
```
professional team photo, office environment,
collaboration, company culture, trust
```

### 开发者工具
```
code in action, terminal aesthetics, API visualization,
developer experience, build something great
```

## 示例提示词

### 示例 1: AI 产品首页
```
A stunning hero banner image for an AI automation product,
wide composition (21:9 aspect ratio),
futuristic tech aesthetic with innovative atmosphere.
Key visual: glowing AI brain/processor as focal point,
positioned on the right using rule of thirds.
Background: deep space gradient with floating particles and data streams.
Color theme: Deep Blue (#0f172a to #1e3a5f) with cyan accents (#22d3ee).
Lighting: dramatic rim lighting on main element, soft ambient glow.
Large empty space on left side for headline text.
Quality: ultra high resolution, 4K, cinematic.
```

### 示例 2: SaaS 着陆页
```
A stunning hero banner image for a workflow automation SaaS,
wide composition (16:9 aspect ratio),
clean modern aesthetic with professional atmosphere.
Key visual: abstract flowing lines representing automation,
connecting various app icons in an elegant pattern.
Background: subtle gradient from white to light gray.
Color theme: Minimal White with blue accent (#3b82f6).
Lighting: soft studio lighting, no harsh shadows.
Center area clear for main headline and CTA button.
Quality: ultra high resolution, crisp vectors, 4K.
```

## 尺寸建议

| 用途 | 比例 | 像素 |
|------|------|------|
| 全屏 Hero | 21:9 | 2560x1080 |
| 标准 Hero | 16:9 | 1920x1080 |
| 移动端 Hero | 4:3 | 1200x900 |
| 社交预览 | 1.91:1 | 1200x628 |

## 响应式注意

```
设计时考虑不同屏幕裁切：
- 桌面端：显示完整 16:9
- 平板端：可能裁切为 4:3
- 移动端：可能裁切为 1:1 或更窄

关键元素放在中心区域，避免被裁切
```
