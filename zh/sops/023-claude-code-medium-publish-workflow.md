# 从本地文档到 Medium 发布：Claude Code 自动化工作流

> 使用 Claude Code + Chrome 浏览器自动化完成技术文档翻译和 Medium 发布的完整工作流

**文档编号**: 023
**日期**: 2025-01-01
**标签**: `Claude Code` `Chrome MCP` `Medium` `自动化发布` `内容工作流`

---

## 概述

本文档记录了使用 Claude Code 结合 Chrome MCP 扩展，实现从本地 Markdown 文档到 Medium 平台发布的自动化工作流。整个流程约 10 分钟完成，其中 Chrome 自动化操作仅需约 5 分钟。

### 核心价值

- **效率提升**: 从手动复制粘贴到全自动化发布
- **跨语言支持**: 自动翻译中文文档为英文
- **浏览器自动化**: 利用已登录的 Chrome 会话，无需 API

---

## 1. 前置条件

### 1.1 环境要求

| 组件 | 要求 | 说明 |
|------|------|------|
| Claude Code | v2.0.73+ | CLI 工具 |
| Chrome 浏览器 | 最新版 | 需安装 Claude in Chrome 扩展 |
| Medium 账号 | 已登录 | Chrome 中保持登录状态 |

### 1.2 Chrome MCP 连接

```bash
# 启动带 Chrome 集成的 Claude Code
claude --chrome

# 或在会话中检查连接状态
/chrome
```

**连接验证**:
```
mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty: true)
```

---

## 2. 工作流程

### 2.1 流程图

```
┌─────────────────────────────────────────────────────────────────┐
│                    Claude Code 自动化发布流程                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │ 读取源   │ -> │ 翻译生成 │ -> │ Chrome   │ -> │ 发布确认 │  │
│  │ 文档     │    │ 英文版   │    │ 自动填充 │    │          │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│       |              |               |               |         │
│    Markdown       Write           MCP 操作        用户确认     │
│    ~1 min        ~3 min          ~5 min          ~1 min       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 执行步骤

#### Step 1: 连接 Chrome 浏览器

```bash
# Claude Code 执行
mcp__claude-in-chrome__tabs_context_mcp(createIfEmpty: true)

# 结果: 成功连接，获取当前标签页信息
```

#### Step 2: 读取源文档

```bash
# 搜索文档
Glob("**/*目标文档*")

# 读取内容
Read("/path/to/source-document.md")
```

#### Step 3: 生成英文版

Claude Code 自动完成:
- 翻译核心内容
- 适配英文读者习惯
- 优化文章结构
- 添加 Introduction/Conclusion

#### Step 4: 导航到 Medium

```bash
mcp__claude-in-chrome__navigate(
  url: "https://medium.com/new-story",
  tabId: <current_tab_id>
)
```

#### Step 5: 输入内容

**操作序列**:
1. 输入标题
2. 输入副标题
3. 分段输入正文
4. 添加代码块

```bash
# 输入操作
mcp__claude-in-chrome__computer(
  action: "type",
  text: "文章内容..."
)
```

#### Step 6: 发布

```bash
# 点击 Publish 按钮
mcp__claude-in-chrome__computer(
  action: "click",
  coordinate: [x, y]
)

# 添加话题标签
# 用户确认后点击 "Publish now"
```

---

## 3. 时间统计

| 阶段 | 耗时 | 说明 |
|------|------|------|
| 连接与准备 | ~1 min | Chrome MCP 连接 |
| 文档读取与翻译 | ~4 min | 生成英文版 |
| Chrome 自动化操作 | ~5 min | 导航、输入、发布 |
| **总计** | **~10 min** | 端到端完成 |

---

## 4. 工具链

```
Claude Code
    |
    +-- Glob (文件搜索)
    +-- Read (读取文档)
    +-- Write (生成英文版)
    +-- Chrome MCP
          |
          +-- tabs_context_mcp (连接浏览器)
          +-- navigate (页面导航)
          +-- read_page (读取页面元素)
          +-- computer (点击/输入/滚动)
          +-- screenshot (截图确认)
```

---

## 5. 最佳实践

### 5.1 成功要点

1. **保持 Chrome 登录状态**: 确保 Medium 账号已登录
2. **分段输入内容**: 长文章分段输入，避免超时
3. **用户确认机制**: 发布前获取用户确认

### 5.2 注意事项

- Medium 的代码块需要手动格式化
- 封面图上传需要额外步骤
- 话题标签建议预先准备

### 5.3 可扩展场景

- 批量发布到多个平台
- 定时发布
- 自动添加 SEO 优化内容

---

## 6. 常见问题

### Q1: Chrome MCP 连接失败？

检查:
1. Claude in Chrome 扩展版本 >= 1.0.36
2. Chrome 浏览器正在运行
3. 尝试 `/chrome` 命令重新连接

### Q2: Medium 要求登录？

确保 Chrome 中已登录 Medium 账号，MCP 会复用已有会话。

### Q3: 内容输入中断？

长文章建议分段输入，每段不超过 500 字。

---

## 参考资源

- [Claude Code 官方文档](https://docs.anthropic.com/claude-code)
- [Chrome MCP 集成指南](https://docs.anthropic.com/claude-code/chrome)
- [Medium 写作指南](https://help.medium.com/hc/en-us/categories/201931128-Publishing)
