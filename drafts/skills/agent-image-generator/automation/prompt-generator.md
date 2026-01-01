# 智能提示词生成器

使用 Claude 推理能力，将用户的简单描述转换为专业级图片生成提示词。

## 转换流程

```
用户输入: "画一张科技风的配图，主题是AI自动化"
                    ↓
           Claude 推理分析
                    ↓
┌─────────────────────────────────────────────────────────────┐
│  1. 解析关键词: 科技风, AI, 自动化                            │
│  2. 匹配模板: tech-style                                    │
│  3. 选择变体: 深空科技 (Deep Space Tech)                     │
│  4. 确定配色: 科技蓝 (Tech Blue)                             │
│  5. 填充细节: 主体、背景、光效、构图                          │
│  6. 组装提示词                                               │
└─────────────────────────────────────────────────────────────┘
                    ↓
完整提示词: "A modern tech-inspired illustration of an AI
automation system, featuring a central processor connected
to workflow nodes, clean geometric shapes, circuit patterns,
glowing neon blue accents on dark gradient background..."
```

## 提示词组装逻辑

```javascript
function generatePrompt(userInput) {
  // 1. 分析用户输入
  const analysis = analyzeInput(userInput);

  // 2. 选择最匹配的模板
  const template = selectTemplate(analysis.keywords);

  // 3. 选择风格变体
  const variant = selectVariant(template, analysis.mood);

  // 4. 确定配色方案
  const colors = selectColorPalette(analysis.colorHints);

  // 5. 组装最终提示词
  const prompt = assemblePrompt({
    template: template.basePrompt,
    subject: analysis.subject,
    style: variant.modifier,
    colorScheme: colors.name,
    aspectRatio: analysis.aspectRatio || '16:9'
  });

  return prompt;
}
```

## 关键词映射

### 风格关键词
```yaml
科技/技术/Tech/编程/代码:
  template: tech-style
  variants: [deep-space, minimal-lines, circuit-pattern]

插画/卡通/可爱:
  template: illustration
  variants: [warm-cozy, vibrant-creative]

商务/专业/企业:
  template: illustration
  variants: [business-pro]

网站/首页/Banner:
  template: hero-banner
  variants: [saas-service, tech-product]
```

### 情绪关键词
```yaml
活力/动感/激情:
  mood: vibrant
  colorHint: warm

专业/严肃/商务:
  mood: professional
  colorHint: corporate

温暖/治愈/舒适:
  mood: cozy
  colorHint: warm

科幻/未来/创新:
  mood: futuristic
  colorHint: tech
```

### 颜色关键词
```yaml
蓝色/科技蓝/冷色:
  palette: tech-blue

橙色/暖色/阳光:
  palette: warm-orange

紫色/梦幻:
  palette: dream-purple

绿色/自然/清新:
  palette: nature-green

黑色/暗色/深色:
  palette: dark-mode
```

## Claude 推理模板

当用户请求生成配图时，使用以下推理流程：

```markdown
## 分析用户需求

用户输入: "{user_input}"

### 1. 提取关键信息
- 主题/主体: {extracted_subject}
- 风格倾向: {style_preference}
- 颜色偏好: {color_preference}
- 用途场景: {use_case}
- 尺寸需求: {aspect_ratio}

### 2. 匹配模板
根据关键词匹配到: {matched_template}
选择变体: {selected_variant}
配色方案: {color_palette}

### 3. 生成提示词
{final_prompt}

### 4. 提示词解释
- 主体描述: ...
- 风格设定: ...
- 颜色配置: ...
- 质量参数: ...
```

## 示例转换

### 示例 1
```
输入: "给我画一张程序员在用AI工具的图"

分析:
- 主体: 程序员使用 AI 工具
- 风格: 技术相关 → tech-style
- 场景: 工作场景
- 颜色: 未指定 → 默认科技蓝

输出:
"A modern tech-inspired illustration of a developer using
AI-powered coding tools, featuring a person at a sleek desk
with multiple holographic interfaces showing code and AI
suggestions, clean geometric shapes, glowing blue accents
on a dark gradient background. Style: deep space aesthetic,
professional. Color scheme: Tech Blue. Quality: 4K, 16:9."
```

### 示例 2
```
输入: "温暖风格的团队协作配图，用在公众号"

分析:
- 主体: 团队协作
- 风格: 温暖 → illustration (warm-cozy)
- 用途: 公众号 → 2.35:1 比例
- 颜色: 温暖 → warm-orange

输出:
"A flat design illustration of a team collaborating together,
four people gathered around a table with warm smiles and
engaged body language, soft rounded shapes, warm orange and
cream color palette with golden highlights. Style: cozy,
friendly, approachable. Background: simple room interior
with plants and sunlight. Quality: vector-style, 900x383."
```

## 质量控制

### 必须包含的元素
- 主体描述 (Subject)
- 风格定义 (Style)
- 颜色方案 (Color scheme)
- 构图说明 (Composition)
- 质量参数 (Quality)

### 可选增强元素
- 光效说明 (Lighting)
- 背景描述 (Background)
- 情绪氛围 (Mood)
- 细节点缀 (Details)

### 格式要求
- 英文提示词（Gemini 对英文支持更好）
- 逻辑清晰，层次分明
- 避免矛盾描述
- 长度适中（50-150 words）
