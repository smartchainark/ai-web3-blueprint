# Article Generation Pipeline: AI-Powered Multi-Style Content Factory

> From source document to multi-platform publishing - a fully automated content production pipeline with parallel generation, AI imaging, and template customization

**Document ID**: 021
**Date**: 2025-12-29
**Tags**: `AI Generation` `Content Pipeline` `Multi-Style` `Image Generation` `Auto-Publishing`

---

## Overview

This document describes a complete AI-driven content generation pipeline that automatically converts a source document into multiple style variations, paired with AI-generated images, template customization, and multi-platform publishing.

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Article Generation Pipeline                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐                                                            │
│  │ Source Doc  │  ← Original Markdown/Document                              │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │              Stage 1: Parallel Multi-Style Generation        │           │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │           │
│  │  │  Technical  │  │ Storytelling│  │   Product   │          │           │
│  │  │   Agent 1   │  │   Agent 2   │  │   Agent 3   │          │           │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │           │
│  └─────────┼────────────────┼────────────────┼──────────────────┘           │
│            ▼                ▼                ▼                              │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │              Stage 2: Quote Extraction & Image Prompts       │           │
│  │  📝 Quote Detection → 🎨 Context Fusion → 💡 Prompt Gen      │           │
│  └──────────────────────────────┬──────────────────────────────┘           │
│                                 ▼                                           │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │              Stage 3: AI Image Generation (Async)            │           │
│  │  ┌─────────────┐  ┌─────────────┐                           │           │
│  │  │   Jimeng    │  │  Banana AI  │  ← Configurable engines   │           │
│  │  └─────────────┘  └─────────────┘                           │           │
│  └──────────────────────────────┬──────────────────────────────┘           │
│                                 ▼                                           │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │              Stage 4: Template Customization                  │           │
│  │  [ Style ] [ Color ] [ Border ] [ Font ]                     │           │
│  │  Presets: [ Minimal | Tech Blue | Warm | Professional ]      │           │
│  └──────────────────────────────┬──────────────────────────────┘           │
│                                 ▼                                           │
│  ┌─────────────────────────────────────────────────────────────┐           │
│  │              Stage 5: Platform Selection & Publishing         │           │
│  │  [ Zhihu ] [ Juejin ] [ WeChat MP ] [ XiaoHongShu ] [ X ]    │           │
│  └─────────────────────────────────────────────────────────────┘           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 1: Multi-Style Parallel Generation

### 1.1 Three Core Styles

| Style | Characteristics | Use Cases | Target Audience |
|-------|----------------|-----------|-----------------|
| **Technical** | Precise, professional, code-heavy | Tech blogs, dev communities | Developers, tech enthusiasts |
| **Storytelling** | Narrative, emotional, scene-based | WeChat, Zhihu, XiaoHongShu | General users, experience seekers |
| **Product** | Value-driven, feature-focused, clear CTA | Product launches, marketing | Potential users, decision makers |

### 1.2 Parallel Generation Architecture

```javascript
// Launch 3 parallel agents using Claude Task tool
async function generateMultiStyleArticles(sourceDoc) {
  const styles = ['technical', 'storytelling', 'product'];

  const tasks = styles.map(style => Task({
    subagent_type: 'general-purpose',
    prompt: `Rewrite the following document in ${style} style:\n\n${sourceDoc}`,
    model: 'sonnet'
  }));

  return Promise.all(tasks);
}
```

### 1.3 Style Transformation Templates

#### Technical Style
- Clear structure with numbered sections
- Code examples for every concept
- Performance data and comparison tables
- Actionable commands and configurations
- Quote format: `> 💡 **Technical Insight**: [content]`

#### Storytelling Style
- Scene-based opening
- First-person narrative perspective
- Emotional connection through pain points
- Story arc: problem → exploration → breakthrough
- Quote format: `> ✨ **Moment of Insight**: [content]`

#### Product Style
- Value proposition upfront
- 3-5 key selling points with icons
- Comparison table vs alternatives
- 3-step quick start guide
- Clear CTA (Call to Action)
- Quote format: `> 🚀 **Core Value**: [content]`

---

## Part 2: Quote Extraction & Image Prompts

### 2.1 Quote Identification Rules

Memorable quotes are the most shareable sentences, typically:
- Concise: 10-30 words
- Opinionated: Express unique insights
- Memorable: Rhythmic, punchy
- Emotional: Resonate with readers
- Valuable: Convey core message

### 2.2 Image Prompt Generation

```javascript
function generateImagePrompt(quote, context) {
  return {
    subject: extractSubject(quote),
    style: {
      art_style: 'digital illustration, modern, clean',
      color_palette: 'vibrant, tech-inspired',
      mood: 'innovative, professional'
    },
    context_elements: extractKeyElements(context),
    technical: {
      aspect_ratio: '16:9',
      quality: 'high detail, 4K'
    }
  };
}
```

---

## Part 3: AI Image Generation

### 3.1 Supported Engines

| Engine | Features | Pricing |
|--------|----------|---------|
| Jimeng | Chinese prompt optimization, fast | Per-image |
| Banana AI | Stable Diffusion, highly customizable | Per-second |
| Midjourney | Highest quality, unique styles | Subscription |

### 3.2 Async Workflow

```
1. Submit image task → Get Task ID
2. Continue other processing (non-blocking)
3. Poll for completion status
4. Replace placeholder when ready
```

---

## Part 4: Template Customization

### 4.1 Preset Themes

- **Minimal White**: Clean, professional
- **Tech Blue**: Modern, innovative
- **Warm Orange**: Friendly, approachable
- **Professional Black**: Sleek, authoritative

### 4.2 Customization Options

- 🎨 Colors: Primary, accent, background
- 📐 Borders: None, rounded, shadow, glow
- ✏️ Fonts: Heading, body, code
- 💬 Quote styles: Left border, card, highlight, bubble
- 🖼️ Image layouts: Full-width, centered, card

---

## Part 5: Platform Publishing

### 5.1 Supported Platforms

| Platform | Status | Special Requirements | Skill |
|----------|--------|---------------------|-------|
| Zhihu | ✅ Ready | Native Markdown | `zhihu-publisher` |
| Juejin | ✅ Ready | Category + Tags required | `juejin-publisher` |
| WeChat MP | ✅ Ready | At least 1 image required | `wechat-publisher` |
| XiaoHongShu | 🔜 Planned | Image-heavy | - |
| X/Twitter | 🔜 Planned | 280 char limit | - |

### 5.2 Multi-Platform Publishing

```javascript
async function publishToMultiplePlatforms(article, platforms) {
  const results = {};
  for (const platform of platforms) {
    const adapted = await adaptForPlatform(article, platform);
    results[platform] = await publish(adapted, platform);
  }
  return results;
}
```

---

## Usage Examples

```bash
# Full pipeline
"Generate 3 style versions from doc 015, add AI images, publish to Zhihu and Juejin"

# Step by step
"Generate technical version"
"Extract quotes and create image prompts"
"Generate images with Jimeng"
"Apply Tech Blue theme"
"Publish to Zhihu"
```

---

## References

- [015 Zhihu Auto-Publish SOP](./015-zhihu-auto-publish-mcp-playwright.md)
- [014 WeChat MP Auto-Publish SOP](./014-wechat-mp-auto-publish-mcp-playwright.md)
- [013 Juejin Auto-Publish SOP](./013-juejin-auto-publish-mcp-playwright.md)
- [Claude Code Skills Guide](https://docs.anthropic.com/claude-code/skills)
