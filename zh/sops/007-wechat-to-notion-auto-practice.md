# 微信文章到 Notion 自动实践工作流

> 从阅读到实践：AI 自动模拟作者操作，沉淀知识库并快速学以致用

**文档编号**: 007
**日期**: 2025-12-29
**标签**: `微信` `Notion` `Claude Code` `MCP` `自动化` `知识管理`

---

## 概述

当你在微信公众号看到一篇技术文章，想要立即实践时，传统方式需要：
1. 阅读文章 → 2. 手动复制命令 → 3. 逐步执行 → 4. 遇到问题再回去看

这个工作流让 **Claude Code 自动模拟作者的所有操作**，实现：
- **即时实践** — 文章内容自动转化为可执行步骤
- **知识沉淀** — 文章自动归档到 Notion 知识库
- **技能固化** — 实践经验可转化为可复用的 Skill

---

## 工作流架构

```
微信公众号文章
      ↓
  Safari 浏览器（外部打开）
      ↓
  分享到 Notion（剪藏）
      ↓
  复制 Notion 链接
      ↓
  Claude Code CLI
      ↓
  MCP 读取 Notion 内容
      ↓
  AI 分析并自动执行安装/配置
      ↓
  ✅ 实践完成 + 知识沉淀
```

---

## 第一部分：环境准备

### 1.1 必要工具

| 工具 | 用途 | 安装状态 |
|------|------|----------|
| Claude Code CLI | AI 编程助手 | `claude --version` |
| Notion MCP | 读取 Notion 内容 | `claude mcp list` |
| Notion 账号 | 知识库存储 | - |
| Safari/Chrome | 外部浏览器 | - |

### 1.2 Notion MCP 配置

如果尚未配置 Notion MCP，执行：

```bash
# 添加 Notion MCP 服务器
claude mcp add notion npx -y @notionhq/notion-mcp-server

# 需要设置 NOTION_TOKEN 环境变量
# 获取方式：Notion Settings → Connections → Develop your own integration
```

详细配置参考 [003 - Cursor IDE 技能与 MCP 配置](./003-cursor-openskills-mcp-setup.md)

---

## 第二部分：完整操作流程

### Step 1: 微信文章 → 外部浏览器

1. 在微信公众号打开文章
2. 点击右上角 `···` 菜单
3. 选择 **"在默认浏览器打开"** 或 **"在 Safari 中打开"**

> **为什么要用外部浏览器？**
> - 微信内置浏览器无法直接分享到 Notion
> - Safari/Chrome 支持 Notion Web Clipper 扩展

### Step 2: 浏览器 → Notion

**方式 A：使用 Notion Web Clipper（推荐）**

1. 安装 [Notion Web Clipper](https://www.notion.so/web-clipper) 浏览器扩展
2. 在文章页面点击扩展图标
3. 选择保存位置（建议创建专门的"文章收藏"数据库）
4. 点击 **Save page**

**方式 B：使用 iOS/macOS 分享菜单**

1. 点击 Safari 分享按钮
2. 选择 **Notion**
3. 选择保存位置并确认

### Step 3: 获取 Notion 链接

1. 打开 Notion，找到刚保存的文章
2. 点击页面右上角 `···` → **Copy link**
3. 或直接从浏览器地址栏复制链接

链接格式示例：
```
https://www.notion.so/Article-Title-2d83f9a673f781d0bb56ed599d8905cb
```

### Step 4: Claude Code 自动实践

将 Notion 链接发送给 Claude Code：

```bash
# 启动 Claude Code
claude

# 发送请求
> 帮我安装这篇文章介绍的工具：https://www.notion.so/xxx
```

Claude Code 会自动：
1. **调用 Notion MCP** 读取文章内容
2. **分析文章结构** 提取安装步骤
3. **执行安装命令** 模拟作者操作
4. **处理异常情况** 自动解决依赖问题
5. **验证安装结果** 确认功能正常

### Step 5: 知识固化（可选）

实践完成后，可以进一步：

**A. 创建 Skill**
```
> 把刚才的安装步骤封装成一个 skill
```

**B. 写入蓝皮书**
```
> 把这个工具的使用方法写成 SOP 文档
```

---

## 第三部分：实际案例

### 案例：安装 claude-mem 记忆插件

**原始文章**：《彻底告别"金鱼记忆"！Claude Code 最强记忆外挂》

**操作过程**：

1. 微信打开文章 → Safari 打开
2. 分享到 Notion 知识库
3. 复制 Notion 链接给 Claude Code：
   ```
   > 帮我安装这个：https://www.notion.so/Claude-Code-2d83f9a673f781d0bb56ed599d8905cb
   ```

4. Claude Code 自动执行：
   ```bash
   # 1. 读取 Notion 页面内容
   # 2. 提取安装命令
   claude plugin marketplace add thedotmack/claude-mem
   claude plugin install claude-mem
   # 3. 验证安装成功
   ```

5. 结果：
   ```
   ✅ claude-mem 安装成功
   ├─ marketplace: thedotmack (已添加)
   └─ plugin: claude-mem@thedotmack (scope: user)
   ```

---

## 第四部分：进阶技巧

### 4.1 批量处理文章

如果有多篇待实践的文章，可以：

```
> 我在 Notion 有一个"待实践"数据库，帮我逐个安装里面的工具
```

### 4.2 自动创建 Skill

实践后自动封装为可复用技能：

```
> 把刚才安装 {工具名} 的步骤封装成 skill，以后一键安装
```

### 4.3 与任务管理集成

结合 task-manager 技能：

```
> 添加任务：实践 claude-mem 记忆插件
```

---

## 常见问题

### Q1: Notion MCP 无法读取页面内容

**原因**：Notion Integration 没有页面访问权限

**解决**：
1. 打开 Notion 页面
2. 点击右上角 `···` → **Connections**
3. 添加你的 Integration

### Q2: 文章内容过长导致解析失败

**解决**：让 Claude Code 分段处理

```
> 先帮我提取这篇文章的安装步骤，然后再执行
```

### Q3: 安装命令过时或有误

**解决**：Claude Code 会自动：
- 检查 GitHub 仓库最新版本
- 查找官方文档验证
- 提示用户确认后再执行

---

## 工作流优势

| 传统方式 | 本工作流 |
|----------|----------|
| 手动复制粘贴命令 | AI 自动提取并执行 |
| 遇到错误需要自己排查 | AI 自动处理异常 |
| 知识分散在各处 | 统一沉淀到 Notion |
| 每次都要重新学习 | 一次实践，封装为 Skill |

---

## 最佳实践

### Do's

- 在 Notion 创建专门的"技术文章"数据库，便于管理
- 实践后立即写笔记或创建 Skill，趁热打铁
- 保留原文链接，方便日后查阅

### Don'ts

- 不要在不信任的环境中自动执行安装命令
- 不要跳过验证步骤，确保安装成功
- 不要忽略文章的前置条件说明

---

## 参考资源

- [Notion Web Clipper](https://www.notion.so/web-clipper)
- [Notion MCP Server](https://github.com/notionhq/notion-mcp-server)
- [003 - Cursor IDE 技能与 MCP 配置](./003-cursor-openskills-mcp-setup.md)
- [006 - Claude 技能创建规范化](./006-claude-skill-creation-sop.md)
