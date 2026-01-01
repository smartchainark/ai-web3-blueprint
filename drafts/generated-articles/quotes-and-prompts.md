# 金句提取与图片提示词生成报告

> 基于 015-知乎自动发布 文章生成的三种风格文案的金句和配图提示词

---

## 一、金句汇总

### 技术型金句 (3条)

| 序号 | 金句 | 类型标记 |
|------|------|----------|
| 1 | 会话持久化是自动化发布的关键，通过复用浏览器 Profile 实现"一次登录，长期有效" | 💡 技术洞察 |
| 2 | 富文本编辑器的 DOM 操作需通过事件系统触发，直接修改 innerHTML 会导致状态不同步 | 💡 技术洞察 |
| 3 | 平台的自动化难度与其编辑器架构直接相关 | 💡 技术洞察 |

### 故事型金句 (3条)

| 序号 | 金句 | 类型标记 |
|------|------|----------|
| 1 | 人生中最消磨意志的，不是困难的挑战，而是日复一日的机械重复 | ✨ 灵感时刻 |
| 2 | 真正的自动化，不是写一堆复杂脚本，而是让机器理解你的意图，一句话就够了 | ✨ 灵感时刻 |
| 3 | 工具的价值，不在于它有多酷炫，而在于它让你把时间还给了真正值得投入的事 | ✨ 灵感时刻 |

### 产品型金句 (3条)

| 序号 | 金句 | 类型标记 |
|------|------|----------|
| 1 | 从复制粘贴到一句话发布，让内容创作者专注于创作本身 | 🚀 核心价值 |
| 2 | 内容创作者每天可节省 1-2 小时发布时间，用于创作更有价值的内容 | 💡 效率革命 |
| 3 | 当你有 10 篇文章需要发布时，手动需要 100 分钟，自动化只需 5 分钟 | 🎯 场景价值 |

---

## 二、图片提示词生成

### 金句 1：会话持久化是自动化发布的关键

```yaml
image_prompt:
  id: quote_tech_01
  quote: "会话持久化是自动化发布的关键"
  style: technical

  prompt_cn: |
    一个现代化的浏览器窗口，显示知乎编辑器界面，
    周围环绕着金色的数据流和连接线，
    象征着会话持久化和自动化连接，
    科技蓝配色，发光效果，
    数字插画风格，简洁现代，4K高清

  prompt_en: |
    Modern browser window showing Zhihu editor interface,
    surrounded by golden data streams and connection lines,
    symbolizing session persistence and automation,
    tech blue color palette, glowing effects,
    digital illustration style, clean and modern, 4K quality

  visual_elements:
    - browser_window
    - data_flow
    - connection_lines
    - glowing_effects

  color_palette: ["#1a73e8", "#4285f4", "#ffc107", "#ffffff"]
  aspect_ratio: "16:9"
  mood: "innovative, professional"
```

### 金句 2：人生中最消磨意志的，是日复一日的机械重复

```yaml
image_prompt:
  id: quote_story_01
  quote: "人生中最消磨意志的，是日复一日的机械重复"
  style: storytelling

  prompt_cn: |
    一个疲惫的创作者坐在电脑前，
    周围漂浮着无数相同的复制粘贴图标，
    形成一个压抑的循环，
    暖色调与冷色调对比，
    表达从困境走向解放的希望，
    插画风格，情感表达，高质量

  prompt_en: |
    A tired creator sitting in front of computer,
    surrounded by floating identical copy-paste icons,
    forming an oppressive cycle,
    warm and cool tone contrast,
    expressing hope from struggle to freedom,
    illustration style, emotional, high quality

  visual_elements:
    - person_at_desk
    - floating_icons
    - circular_pattern
    - lighting_contrast

  color_palette: ["#2c3e50", "#e74c3c", "#f39c12", "#ecf0f1"]
  aspect_ratio: "16:9"
  mood: "contemplative, hopeful"
```

### 金句 3：真正的自动化，是让机器理解你的意图

```yaml
image_prompt:
  id: quote_story_02
  quote: "真正的自动化，是让机器理解你的意图"
  style: storytelling

  prompt_cn: |
    一个人对着AI助手说话，
    话语变成金色的光线连接到电脑屏幕，
    屏幕上显示自动执行的任务流程，
    温暖的光效，未来感设计，
    人机协作的和谐画面，
    3D渲染风格，高质量

  prompt_en: |
    A person speaking to AI assistant,
    words transforming into golden light connecting to screen,
    screen showing automated task workflow,
    warm lighting, futuristic design,
    harmonious human-AI collaboration,
    3D render style, high quality

  visual_elements:
    - human_figure
    - ai_interface
    - speech_to_action
    - task_automation

  color_palette: ["#6366f1", "#fbbf24", "#10b981", "#ffffff"]
  aspect_ratio: "16:9"
  mood: "innovative, harmonious"
```

### 金句 4：从复制粘贴到一句话发布

```yaml
image_prompt:
  id: quote_product_01
  quote: "从复制粘贴到一句话发布"
  style: product

  prompt_cn: |
    左右对比图：
    左侧是混乱的多窗口、复制粘贴箭头、疲惫的表情符号，
    右侧是简洁的单一界面、一句话命令、轻松的表情符号，
    中间用一道光芒分隔，象征转变，
    产品风格，对比鲜明，专业设计

  prompt_en: |
    Before and after comparison:
    Left side: chaotic multiple windows, copy-paste arrows, tired emoji,
    Right side: clean single interface, one-line command, relaxed emoji,
    divided by a beam of light symbolizing transformation,
    product style, high contrast, professional design

  visual_elements:
    - split_screen
    - before_after
    - transformation_light
    - emoji_expressions

  color_palette: ["#ef4444", "#22c55e", "#f8fafc", "#1e293b"]
  aspect_ratio: "16:9"
  mood: "transformative, empowering"
```

### 金句 5：工具的价值，在于它让你把时间还给真正值得投入的事

```yaml
image_prompt:
  id: quote_story_03
  quote: "工具的价值，在于它让你把时间还给真正值得投入的事"
  style: storytelling

  prompt_cn: |
    一个创作者从电脑前站起来，
    身后的自动化机器人在处理发布任务，
    创作者面前是画布、书籍、创意工具，
    温暖的阳光洒进房间，
    表达从繁琐工作中解放后的自由创作，
    插画风格，温馨治愈，高质量

  prompt_en: |
    A creator standing up from computer,
    automation robot handling publishing tasks behind,
    facing canvas, books, creative tools in front,
    warm sunlight filling the room,
    expressing freedom to create after liberation from tedious work,
    illustration style, warm and healing, high quality

  visual_elements:
    - person_freedom
    - robot_assistant
    - creative_tools
    - warm_lighting

  color_palette: ["#fbbf24", "#f97316", "#84cc16", "#fef3c7"]
  aspect_ratio: "16:9"
  mood: "liberating, inspiring"
```

---

## 三、批量生图任务

```javascript
// 生图任务队列
const imageGenerationTasks = [
  {
    id: 'quote_tech_01',
    engine: 'jimeng', // 或 'banana'
    prompt: '现代化浏览器窗口，知乎编辑器界面，金色数据流...',
    style: 'digital_illustration',
    size: '1024x576',
    priority: 1
  },
  {
    id: 'quote_story_01',
    engine: 'jimeng',
    prompt: '疲惫创作者，复制粘贴图标循环...',
    style: 'illustration',
    size: '1024x576',
    priority: 2
  },
  {
    id: 'quote_story_02',
    engine: 'jimeng',
    prompt: '人机对话，话语变光线，自动化流程...',
    style: '3d_render',
    size: '1024x576',
    priority: 2
  },
  {
    id: 'quote_product_01',
    engine: 'jimeng',
    prompt: '左右对比图，混乱到简洁的转变...',
    style: 'product',
    size: '1024x576',
    priority: 1
  },
  {
    id: 'quote_story_03',
    engine: 'jimeng',
    prompt: '创作者解放，机器人处理任务，创意工具...',
    style: 'illustration',
    size: '1024x576',
    priority: 3
  }
];

// 异步执行生图任务
async function generateAllImages() {
  const results = await Promise.all(
    imageGenerationTasks.map(task =>
      generateImage(task.prompt, task.style, task.size)
    )
  );
  return results;
}
```

---

## 四、图片插入位置映射

```yaml
article_image_mapping:
  technical_style:
    - position: "after_section_1.1"
      quote_id: quote_tech_01
      caption: "会话持久化架构示意图"

  storytelling_style:
    - position: "after_paragraph_2"
      quote_id: quote_story_01
      caption: "重复劳动的困境"
    - position: "after_paragraph_6"
      quote_id: quote_story_02
      caption: "一句话的魔力"
    - position: "before_conclusion"
      quote_id: quote_story_03
      caption: "真正的解放"

  product_style:
    - position: "hero_image"
      quote_id: quote_product_01
      caption: "从繁琐到简洁的转变"
```

---

**生成时间**: 2025-12-29
**关联文章**: 016-article-generation-pipeline.md
**状态**: 待生图引擎处理
