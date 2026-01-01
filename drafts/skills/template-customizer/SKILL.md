---
name: template-customizer
description: 文章模板定制器，支持风格、颜色、边框、字体等二次加工选项。当用户说"选择模板"、"定制样式"、"customize template"、"换个风格"时触发此技能。
---

# 模板定制器 (Template Customizer)

为生成的文章进行二次加工，选择风格、颜色、边框等视觉元素。

## 预设主题

### 简约白 (Minimal White)
```yaml
theme:
  primary_color: '#ffffff'
  accent_color: '#333333'
  background: 'linear-gradient(180deg, #fff 0%, #f5f5f5 100%)'
typography:
  heading: 'PingFang SC'
  body: 'Noto Sans SC'
elements:
  border: 'none'
  quote_style: 'left-border'
  code_theme: 'github-light'
```

### 科技蓝 (Tech Blue)
```yaml
theme:
  primary_color: '#1a73e8'
  accent_color: '#4285f4'
  background: 'linear-gradient(135deg, #1a1a2e 0%, #16213e 100%)'
typography:
  heading: 'SF Pro Display'
  body: 'Inter'
elements:
  border: 'glow'
  quote_style: 'card'
  code_theme: 'one-dark'
```

### 暖心橙 (Warm Orange)
```yaml
theme:
  primary_color: '#ff6b35'
  accent_color: '#f7c59f'
  background: 'linear-gradient(180deg, #fff5eb 0%, #ffe4d6 100%)'
typography:
  heading: 'Noto Serif SC'
  body: 'Noto Sans SC'
elements:
  border: 'rounded'
  quote_style: 'bubble'
```

### 专业黑 (Professional Black)
```yaml
theme:
  primary_color: '#000000'
  accent_color: '#666666'
  background: '#121212'
typography:
  heading: 'Roboto'
  body: 'Source Sans Pro'
elements:
  border: 'sharp'
  quote_style: 'highlight'
  code_theme: 'dracula'
```

## 可定制项

### 🎨 颜色
- 主色调
- 强调色
- 背景色/渐变

### 📐 边框
- 无边框
- 圆角
- 阴影
- 发光效果

### ✏️ 字体
- 标题字体
- 正文字体
- 代码字体

### 💬 金句样式
- 左边框
- 卡片式
- 高亮背景
- 气泡框

### 🖼️ 图片布局
- 全宽
- 居中
- 左对齐
- 卡片式

## 使用示例

```
# 选择预设主题
使用科技蓝主题

# 自定义组合
选择圆角边框 + 卡片式金句

# 完整定制
主色调 #6366f1，暖色背景，发光边框
```

## 交互式选择

```
🎨 选择主题风格
  ○ 简约白  ● 科技蓝  ○ 暖心橙  ○ 专业黑

🖼️ 边框样式
  [ ] 无边框  [✓] 圆角  [ ] 阴影  [ ] 发光

💬 金句样式
  ○ 左边框  ● 卡片式  ○ 高亮背景  ○ 气泡框

[预览效果]  [保存配置]  [应用并继续]
```

## 输出格式

根据目标平台自动调整：
- **知乎**：Markdown + 内联样式
- **微信**：HTML 富文本
- **小红书**：图片渲染

## 集成能力

- 与 `article-generator` 联动
- 配置可保存为预设
- 支持批量应用到多篇文章
