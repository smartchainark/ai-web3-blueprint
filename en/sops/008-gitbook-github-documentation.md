# GitBook + GitHub Documentation Workflow

> Standardized bilingual documentation management using GitBook structure + GitHub hosting.

**Document ID**: 008
**Date**: 2025-12-29
**Tags**: `GitBook` `GitHub` `Documentation` `Bilingual` `HonKit`

---

## Overview

This SOP covers how to combine GitBook directory structure with GitHub repositories to achieve:

- Standardized bilingual documentation management
- Local preview and online publishing
- Version control and collaboration

## Directory Structure

```
project/
├── book.json           # GitBook configuration
├── SUMMARY.md          # Root navigation (multi-language entry)
├── README.md           # English homepage + GitHub display
├── README.zh.md        # Chinese homepage
├── en/                 # English docs
│   ├── README.md       # English overview
│   ├── SUMMARY.md      # English navigation
│   ├── sops/           # SOP documents
│   ├── thoughts/       # Essays
│   └── resources/      # Resources (glossary, tools)
├── zh/                 # Chinese docs
│   ├── README.md       # Chinese overview
│   ├── SUMMARY.md      # Chinese navigation
│   ├── sops/
│   ├── thoughts/
│   └── resources/
└── .gitignore          # Ignore _book/ build artifacts
```

## Core Files

### 1. book.json

GitBook/HonKit configuration file:

```json
{
  "title": "Project Name",
  "description": "Project description",
  "author": "Author",
  "language": "en",
  "structure": {
    "readme": "README.md",
    "summary": "SUMMARY.md"
  },
  "links": {
    "sidebar": {
      "GitHub": "https://github.com/your/repo"
    }
  },
  "plugins": ["-sharing"]
}
```

**Notes**:
- `plugins: ["-sharing"]` disables share buttons to avoid rendering issues
- Complex plugins may cause HonKit errors, keep it simple

### 2. SUMMARY.md (Root)

Multi-language navigation entry:

```markdown
# Summary

* [Introduction](README.md)

## English

* [Overview](en/README.md)
* [SOPs & Guides](en/sops/001-first-doc.md)
  * [First Doc](en/sops/001-first-doc.md)
  * [Second Doc](en/sops/002-second-doc.md)
* [Thoughts](en/thoughts/README.md)
* [Glossary](en/resources/glossary.md)

## 中文

* [概览](zh/README.md)
* [SOP 与指南](zh/sops/001-first-doc.md)
  * [第一篇](zh/sops/001-first-doc.md)
  * [第二篇](zh/sops/002-second-doc.md)
* [思考随笔](zh/thoughts/README.md)
* [术语表](zh/resources/glossary.md)
```

### 3. README.md (GitHub Homepage)

Serves as both GitHub repo homepage and GitBook entry:

```markdown
# Project Name

[![License](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)]()
![GitHub stars](https://img.shields.io/github/stars/user/repo?style=social)

> Project description

[中文版](./README.zh.md)

---

## Documentation Index

| # | Title | Description | Date |
|---|-------|-------------|------|
| 001 | [Doc Title](./en/sops/001-doc.md) | Description | 2025-12-29 |

---

**Author**: name
**GitHub**: https://github.com/user/repo
```

## Workflow

### Step 1: Initialize Project

```bash
# Create directory structure
mkdir -p en/sops en/thoughts en/resources
mkdir -p zh/sops zh/thoughts zh/resources

# Create core files
touch book.json SUMMARY.md README.md README.zh.md
touch en/README.md en/SUMMARY.md
touch zh/README.md zh/SUMMARY.md
```

### Step 2: Install HonKit

Since `gitbook-cli` is incompatible with Node.js 18+, use HonKit instead:

```bash
# Global installation
npm install -g honkit

# Or local project installation
npm install honkit --save-dev
```

### Step 3: Local Preview

```bash
# Start development server
honkit serve

# Access http://localhost:4000
```

### Step 4: Build Static Files

```bash
# Build to _book/ directory
honkit build

# Check output
ls _book/
```

### Step 5: Git Version Control

```bash
# Add build artifacts to .gitignore
echo "_book/" >> .gitignore
echo "node_modules/" >> .gitignore

# Commit code
git add .
git commit -m "docs: add new documentation"
git push
```

## Document Naming Conventions

### File Names

```
{number}-{english-title-lowercase-hyphenated}.md

Examples:
001-notion-mcp-setup.md
002-internal-comms-to-exports.md
008-gitbook-github-documentation.md
```

### Document Structure

```markdown
# Title

> One-line description

**Document ID**: NNN
**Date**: YYYY-MM-DD
**Tags**: `tag1` `tag2`

---

## Overview

...

## Step 1: XXX

...

## FAQ

...

## References

- [Link](url)
```

## Publishing Options

### Option 1: GitHub Pages

```bash
# Install gh-pages plugin
npm install gh-pages --save-dev

# Deploy to GitHub Pages
npx gh-pages -d _book
```

### Option 2: GitBook.com (Recommended)

GitBook.com provides hosting with automatic GitHub sync - no manual builds required.

#### Step 1: Create GitBook Account and Space

1. Visit [gitbook.com](https://www.gitbook.com) and sign up
2. Create a new Organization (or use personal space)
3. Click **"Create a space"** to create a documentation space
4. Enter space name (e.g., "SOP Blueprint")

#### Step 2: Connect GitHub Repository

1. Go to Space settings → **Git Sync**
2. Click **"Connect with GitHub"**
3. Authorize GitBook to access your GitHub account
4. Select target repository (e.g., `smartchainark/ai-web3-blueprint`)
5. Configure sync options:
   - **Branch**: `main` (or your main branch)
   - **Project directory**: `./` (root, since SUMMARY.md is in root)

#### Step 3: Configure Monorepo (Optional)

If your docs are in a subdirectory, configure **Project directory**:

| Scenario | Project directory setting |
|----------|---------------------------|
| SUMMARY.md in root | `./` |
| Docs in docs/ subdirectory | `./docs` |
| Docs in documentation/ | `./documentation` |

#### Step 4: Publish Site

1. Return to Space homepage
2. Click **"Publish"** button in top right
3. Or click **"Publish your site"** in the **Get started** list
4. GitBook will generate a default domain:
   ```
   https://your-org.gitbook.io/space-name/
   ```

#### Step 5: Configure Custom Domain (Optional)

1. Go to Space settings → **Custom domain**
2. Enter your domain (e.g., `docs.example.com`)
3. Add CNAME record at your DNS provider:
   ```
   docs.example.com → hosting.gitbook.io
   ```
4. Wait for DNS propagation (usually minutes to 24 hours)
5. GitBook will auto-configure SSL certificate

#### Auto-Sync Mechanism

Once configured, every `git push` to GitHub triggers:

```
git push → GitHub receives → GitBook pulls → Site updates
```

**Sync delay**: Usually completes within 1-2 minutes

#### GitBook.com Free Tier Limits

| Feature | Free | Paid |
|---------|------|------|
| Public Spaces | ✅ Unlimited | ✅ Unlimited |
| Private Spaces | 1 | Unlimited |
| Custom Domain | ✅ | ✅ |
| Team Members | 1 | Unlimited |
| Git Sync | ✅ | ✅ |

### Option 3: Static Hosting

Deploy `_book/` directory to any static hosting service:
- Vercel
- Netlify
- Cloudflare Pages

## FAQ

### Q: HonKit throws "Maximum call stack size exceeded"

**A**: Usually a plugin issue, simplify `book.json`:

```json
{
  "plugins": ["-sharing"]
}
```

### Q: gitbook-cli installation fails

**A**: `gitbook-cli` is incompatible with Node.js 18+, use HonKit instead:

```bash
npm uninstall -g gitbook-cli
npm install -g honkit
```

### Q: Chinese filenames appear as links in Telegram

**A**: Telegram recognizes `.md` endings as links. Remove extension in notifications:

```
Files: zh/sops/008-gitbook-github-documentation
```

## References

- [HonKit Documentation](https://github.com/honkit/honkit)
- [GitBook Legacy Docs](https://github.com/GitbookIO/gitbook)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
