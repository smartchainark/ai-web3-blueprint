# Agent 智能生图使用示例

## 快速开始

### 最简单的用法

```
生成一张配图：AI自动化工作流
```

Claude 会自动：
1. 分析"AI自动化工作流"关键词
2. 选择"tech-style"模板
3. 生成专业级英文提示词
4. 打开 Gemini 生图
5. 下载到 `./images/` 目录

---

## 基础示例

### 1. 科技风配图

```
画一张科技风的图，主题是程序员使用AI编程
```

**生成的提示词**:
```
A modern tech-inspired illustration of a developer using
AI-powered coding tools, featuring a person at a sleek desk
with holographic code interfaces and AI assistant visualization,
clean geometric shapes, glowing blue neon accents on dark
gradient background. Style: deep space aesthetic, professional.
Color scheme: Tech Blue. Quality: 4K, 16:9.
```

**输出文件**: `./images/20251229_143052_developer_ai_coding.png`

---

### 2. 扁平插画风格

```
生成一张温暖风格的团队协作插画，用于公众号文章
```

**生成的提示词**:
```
A flat design illustration of a diverse team collaborating,
four people gathered around a table with documents and devices,
warm orange and cream color palette, soft rounded shapes,
friendly expressions and engaged body language.
Style: warm & cozy, approachable.
Background: simple office with plants and sunlight.
Quality: vector-style, 900x383 for WeChat article.
```

**输出文件**: `./images/20251229_144215_team_collaboration.png`

---

### 3. 网站 Hero Banner

```
生成网站首页大图，主题是SaaS自动化产品，需要左侧留白放文字
```

**生成的提示词**:
```
A stunning hero banner for a SaaS automation product,
wide composition (21:9 aspect ratio),
abstract flowing lines connecting various app icons elegantly,
gradient background from deep blue to purple.
Key visual on right side using rule of thirds.
Large empty space on left for headline text overlay.
Style: modern, professional, innovative.
Quality: ultra high resolution, 2560x1080, 4K.
```

**输出文件**: `./images/20251229_145030_saas_hero_banner.png`

---

## 高级示例

### 4. 指定详细参数

```
生成配图，参数如下：
- 主题：云计算架构
- 风格：电路纹理科技风
- 配色：极客紫
- 比例：16:9
- 用途：技术博客封面
```

**生成的提示词**:
```
A modern tech-inspired illustration of cloud computing architecture,
featuring server nodes connected by glowing data streams,
circuit board texture background with flowing particles,
purple gradient (Geek Purple: #6366f1, #8b5cf6, #c084fc).
Style: circuit pattern, tech infrastructure visualization.
Composition: balanced with clear focal point on central server cluster.
Quality: high resolution, 1920x1080, crisp details.
```

---

### 5. 结合文章上下文

```
我正在写一篇关于 MCP Playwright 自动发布的文章，帮我生成一张配图，
要体现"自动化"和"一键发布"的概念
```

**Claude 分析**:
- 文章主题：MCP Playwright 自动发布
- 关键概念：自动化、一键发布
- 视觉隐喻：机器人/自动臂 → 发布按钮 → 多平台

**生成的提示词**:
```
A modern tech-inspired illustration of automated content publishing,
featuring a central "publish" button with robotic arms extending
to multiple platform icons (social media, blog, etc.),
conveyor belt aesthetic with documents flowing smoothly,
clean geometric shapes, glowing blue accents.
Style: automation workflow, streamlined process.
Color scheme: Tech Blue with green success highlights.
Quality: 4K, 16:9, professional.
```

---

### 6. 批量生成系列图

```
为以下3个金句生成系列配图，保持风格统一：
1. "会话持久化是自动化发布的关键"
2. "真正的自动化是让机器理解你的意图"
3. "工具的价值在于让你专注于真正重要的事"
```

**输出**:
```
./images/
├── 20251229_150101_session_persistence.png   # 金句1配图
├── 20251229_150215_ai_understanding.png      # 金句2配图
└── 20251229_150330_tool_value_focus.png      # 金句3配图
```

---

## 特殊场景

### 7. 小红书竖版图

```
生成小红书风格的图，主题是效率提升，3:4竖版
```

**生成的提示词**:
```
A vibrant flat illustration of productivity boost concept,
person surrounded by floating task checkmarks and achievement badges,
warm gradient background (peach to coral),
bold colors, playful style, XiaoHongShu aesthetic.
Composition: vertical 3:4, content in upper 2/3 for text space below.
Quality: 1080x1440, Instagram-ready.
```

---

### 8. 暗色模式友好

```
生成深色背景的配图，适合暗色模式网站，主题是API接口
```

**生成的提示词**:
```
A modern tech illustration of API interface connections,
multiple endpoints linked by glowing data lines,
dark background (#18181b) with bright accent lines,
code snippets floating in 3D space.
Style: dark mode friendly, minimal, professional.
Color scheme: Dark Mode with gold accents (#fbbf24).
Quality: 4K, 16:9, high contrast for dark backgrounds.
```

---

## 常用命令速查

| 需求 | 命令 |
|------|------|
| 简单生图 | `生成配图：[主题]` |
| 指定风格 | `生成科技风配图：[主题]` |
| 指定用途 | `生成博客封面：[主题]` |
| 指定比例 | `生成16:9横版图：[主题]` |
| 指定配色 | `使用蓝色主题生成：[主题]` |
| 批量生成 | `为这3个标题各生成配图` |
| 结合上下文 | `为这篇文章生成配图` |

---

## 输出说明

所有图片默认保存到当前目录的 `./images/` 文件夹：

```bash
# 查看生成的图片
ls -la ./images/

# 输出示例
-rw-r--r--  1 user  staff  245KB  Dec 29 15:01  20251229_150101_ai_automation.png
-rw-r--r--  1 user  staff  312KB  Dec 29 15:02  20251229_150215_team_work.png
-rw-r--r--  1 user  staff  198KB  Dec 29 15:03  20251229_150330_hero_banner.png
```

---

## 注意事项

1. **首次使用需登录**: 第一次使用会打开 Gemini，需要登录 Google 账号
2. **登录后持久化**: 登录一次后会保持登录状态
3. **生成时间**: 每张图大约需要 10-30 秒
4. **免费额度**: Gemini 免费版有每日生成限制
5. **内容政策**: 遵守 Google 内容生成政策
