# Claude Skills 跨平台部署与管理指南

> 跨平台复用的 AI Skill 工作环境 — 一次配置，多设备同步，让你的 AI 技能随身携带

**文档编号**: 004
**日期**: 2025-12-29
**标签**: `Claude Code` `Skills` `跨平台` `Git`

---

## 概述

本文档演示如何部署和管理 Claude Code 的技能系统，实现跨平台复用和多设备同步：

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1. Clone 仓库 | `git clone` | 获取技能库 |
| 2. 创建符号链接 | `ln -s` | 连接到 Claude |
| 3. 启用技能 | 符号链接 | 按需启用 |
| 4. 同步更新 | `git pull` | 多设备同步 |

```
Claude Code Runtime
       ↓
~/.claude/skills/ (系统读取位置)
       ↓ [符号链接]
my_skils/enabled/ (已启用技能)
       ↓ [符号链接]
my_skils/library/ (全部技能库)
       ↓
GitHub Repository (版本控制)
```

**适用环境**：macOS / Linux / Windows
**预计完成时间**：10-15 分钟

---

## 第一部分：架构理解

### 1.1 核心概念

| 目录 | 作用 |
|------|------|
| `~/.claude/skills/` | Claude Code 读取技能的系统目录 |
| `my_skils/enabled/` | 已启用技能目录（符号链接集合） |
| `my_skils/library/` | 全部技能库（实际文件） |

### 1.2 设计优势

1. **单一数据源** — 所有技能只存一份（在 `library/`）
2. **灵活控制** — 通过符号链接控制启用/禁用
3. **版本管理** — 整个仓库可 Git 管理
4. **跨平台** — 符号链接在 macOS/Linux/Windows 都支持

---

## 第二部分：首次部署

### 2.1 macOS / Linux

```bash
# Step 1: Clone 仓库
git clone git@github.com:smartchainark/my_skils.git ~/Projects/my_skils

# Step 2: 备份现有技能（如有）
[ -d ~/.claude/skills ] && mv ~/.claude/skills ~/.claude/skills.bak

# Step 3: 创建符号链接
ln -s ~/Projects/my_skils/enabled ~/.claude/skills

# Step 4: 验证
ls -la ~/.claude/skills
# 应显示: ~/.claude/skills -> ~/Projects/my_skils/enabled
```

### 2.2 Windows (CMD - 管理员)

```cmd
REM Step 1: Clone 仓库
git clone git@github.com:smartchainark/my_skils.git D:\Projects\my_skils

REM Step 2: 备份现有技能（如有）
if exist %USERPROFILE%\.claude\skills move %USERPROFILE%\.claude\skills %USERPROFILE%\.claude\skills.bak

REM Step 3: 创建 Junction（推荐，无需管理员）
mklink /J %USERPROFILE%\.claude\skills D:\Projects\my_skils\enabled

REM 或创建符号链接（需要管理员或开发者模式）
mklink /D %USERPROFILE%\.claude\skills D:\Projects\my_skils\enabled
```

### 2.3 Windows (PowerShell)

```powershell
# Step 1: Clone 仓库
git clone git@github.com:smartchainark/my_skils.git D:\Projects\my_skils

# Step 2: 备份现有技能
if (Test-Path "$env:USERPROFILE\.claude\skills") {
    Move-Item "$env:USERPROFILE\.claude\skills" "$env:USERPROFILE\.claude\skills.bak"
}

# Step 3: 创建 Junction
cmd /c mklink /J "$env:USERPROFILE\.claude\skills" "D:\Projects\my_skils\enabled"
```

### 2.4 Windows 开发者模式设置

如需使用符号链接（而非 Junction）：

1. 打开 **设置** → **系统** → **开发者选项**
2. 开启 **开发者模式**
3. 配置 Git：`git config --global core.symlinks true`

---

## 第三部分：日常使用

### 3.1 启用技能

```bash
cd ~/Projects/my_skils/enabled
ln -s ../library/skill-name .

# 示例：启用 blockchain-developer
ln -s ../library/blockchain-developer .
```

**Windows:**
```cmd
cd D:\Projects\my_skils\enabled
mklink /D blockchain-developer ..\library\blockchain-developer
```

### 3.2 禁用技能

```bash
rm ~/Projects/my_skils/enabled/skill-name

# 示例：禁用 blockchain-developer
rm ~/Projects/my_skils/enabled/blockchain-developer
```

**Windows:**
```cmd
rmdir D:\Projects\my_skils\enabled\blockchain-developer
```

### 3.3 查看技能状态

```bash
# 查看所有可用技能
ls ~/Projects/my_skils/library/

# 查看已启用技能
ls ~/Projects/my_skils/enabled/

# 查看技能详情
cat ~/Projects/my_skils/library/skill-name/SKILL.md
```

### 3.4 快捷命令

添加到 `~/.bashrc` 或 `~/.zshrc`：

```bash
alias skills-list='ls ~/Projects/my_skils/library/'
alias skills-enabled='ls ~/Projects/my_skils/enabled/'
alias skills-enable='cd ~/Projects/my_skils/enabled && ln -s ../library/'
alias skills-disable='rm ~/Projects/my_skils/enabled/'

# 使用
skills-list                        # 列出所有技能
skills-enabled                     # 列出已启用
skills-enable blockchain-developer # 启用技能
skills-disable blockchain-developer # 禁用技能
```

---

## 第四部分：技能管理

### 4.1 添加新技能

```bash
# Step 1: 创建技能目录
mkdir -p ~/Projects/my_skils/library/my-new-skill

# Step 2: 创建 SKILL.md
cat > ~/Projects/my_skils/library/my-new-skill/SKILL.md << 'EOF'
---
name: my-new-skill
description: 技能简短描述
---

# My New Skill

技能详细说明...
EOF

# Step 3: 启用技能
cd ~/Projects/my_skils/enabled
ln -s ../library/my-new-skill .

# Step 4: 提交到 Git
cd ~/Projects/my_skils
git add library/my-new-skill enabled/my-new-skill
git commit -m "feat: add my-new-skill"
git push
```

### 4.2 更新技能

```bash
# 直接编辑
vim ~/Projects/my_skils/library/skill-name/SKILL.md

# 提交更新
cd ~/Projects/my_skils
git add .
git commit -m "update: improve skill-name"
git push
```

### 4.3 删除技能

```bash
# Step 1: 先禁用
rm ~/Projects/my_skils/enabled/skill-name

# Step 2: 删除源文件
rm -rf ~/Projects/my_skils/library/skill-name

# Step 3: 提交
cd ~/Projects/my_skils
git add .
git commit -m "remove: delete skill-name"
git push
```

### 4.4 批量启用技能

```bash
cd ~/Projects/my_skils/enabled

# 启用多个技能
for skill in task-manager xlsx pdf docx; do
    ln -s ../library/$skill .
done
```

### 4.5 重置为默认启用

```bash
cd ~/Projects/my_skils/enabled

# 清空所有启用
rm -f *

# 启用默认技能
for skill in doc-coauthoring docx internal-comms pdf pptx xlsx \
             notion-knowledge-capture notion-research-documentation \
             notion-spec-to-implementation notebooklm task-manager; do
    ln -s ../library/$skill .
done
```

---

## 第五部分：跨平台部署

### 5.1 新设备部署清单

| Step | macOS/Linux | Windows |
|------|-------------|---------|
| 1 | `git clone ...` | `git clone ...` |
| 2 | `ln -s .../enabled ~/.claude/skills` | `mklink /J ...` |
| 3 | 完成 | 完成 |

### 5.2 同步更新

```bash
cd ~/Projects/my_skils
git pull
# 符号链接自动指向最新内容，无需额外操作
```

### 5.3 多设备不同配置

如果不同设备需要启用不同技能：

```bash
# 方案 1: 使用 Git 分支
git checkout -b device-macbook
# 调整 enabled/ 内容
git commit -am "macbook config"

# 方案 2: 本地忽略 enabled/
echo "enabled/" >> .git/info/exclude
# enabled/ 变更不会被跟踪
```

---

## 常见问题

### Q1: 技能不生效

**检查步骤**：

```bash
# 检查 1: 系统链接是否正确
ls -la ~/.claude/skills
# 应显示: ~/.claude/skills -> .../my_skils/enabled

# 检查 2: enabled 内链接是否正确
ls -la ~/Projects/my_skils/enabled/
# 应显示各技能指向 ../library/xxx

# 检查 3: 技能文件是否存在
cat ~/Projects/my_skils/library/skill-name/SKILL.md
```

### Q2: 符号链接损坏

```bash
# 查找损坏的链接
find ~/Projects/my_skils/enabled -type l ! -exec test -e {} \; -print

# 修复：删除损坏链接并重建
rm ~/Projects/my_skils/enabled/broken-link
cd ~/Projects/my_skils/enabled
ln -s ../library/skill-name .
```

### Q3: Windows 符号链接失败

```cmd
REM 使用 Junction 替代（无需管理员权限）
mklink /J target source

REM 或启用开发者模式后重试
```

### Q4: Git 不跟踪符号链接

```bash
# 检查 Git 配置
git config core.symlinks

# 设置为 true
git config --global core.symlinks true

# 重新 clone
rm -rf my_skils
git clone git@github.com:smartchainark/my_skils.git
```

---

## 最佳实践

### Do's

- ✅ 技能只在 `library/` 存一份
- ✅ 通过符号链接控制启用状态
- ✅ 定期 `git pull` 同步更新
- ✅ 新技能先测试再启用
- ✅ 保持 SKILL.md 格式规范

### Don'ts

- ❌ 不要直接在 `enabled/` 创建真实目录
- ❌ 不要在 `~/.claude/skills/` 手动放文件
- ❌ 不要删除 `library/` 中正在使用的技能
- ❌ 不要忽略 Git 提交（保持同步）

### 技能命名规范

```
✓ task-manager          # 小写，连字符分隔
✓ web3-frontend-scaffold
✗ TaskManager           # 不要用驼峰
✗ task_manager          # 不要用下划线
```

### 目录结构规范

```
library/
└── skill-name/
    ├── SKILL.md        # 必须，主技能文件
    ├── templates/      # 可选，模板文件
    ├── examples/       # 可选，示例
    └── data/           # 可选，数据文件
```

---

## 快速参考

| 操作 | 命令 |
|------|------|
| 列出所有技能 | `ls ~/Projects/my_skils/library/` |
| 列出已启用 | `ls ~/Projects/my_skils/enabled/` |
| 启用技能 | `cd enabled && ln -s ../library/xxx .` |
| 禁用技能 | `rm enabled/xxx` |
| 同步更新 | `git pull` |
| 添加技能 | `mkdir library/xxx && vim library/xxx/SKILL.md` |

---

## 参考资源

- [Claude Code Skills 文档](https://docs.anthropic.com/claude-code/skills)
- [GitHub 仓库 - my_skils](https://github.com/smartchainark/my_skils)
