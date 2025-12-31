# From Development Conversations to Reusable Assets

> Transform daily development discussions, decisions, and lessons learned into reusable skills and documentation

**Document ID**: 019
**Date**: 2026-01-01
**Tags**: `knowledge-management` `skill-creation` `asset-building` `best-practices`

---

## Overview

Development processes generate massive amounts of valuable information: requirement discussions, design decisions, bug fixes, code reviews... If this information only exists in chat logs and personal memory, it dissipates over time.

This SOP defines a methodology: **Identify depositworthy content → Abstract into reusable form → Create team assets**.

Core principle: **Today's conversation = Tomorrow's asset**

---

## Part 1: Identifying Signals Worth Capturing

### What Conversations Are Worth Depositing?

| Signal | Description | Example |
|--------|-------------|---------|
| **Recurring** | Same question asked/solved 2+ times | "How do I call this API?" asked 3 times |
| **Pitfall encountered** | Problem that took time to debug | A bug that took 2 hours to locate |
| **Decision made** | Why choose A over B | Reasons for choosing React Query over SWR |
| **Repeated feedback** | Same issue pointed out in code reviews | "Use `Math.floor` not `Math.ceil` here" |
| **Effective workaround** | Temporary solution that proved stable | A workaround running stable for 2 weeks |

### When to Identify

- **During development**: Record immediately after solving a problem
- **During review**: Mark when spotting repeated issues
- **During retrospective**: Review after project completion
- **When asked**: The second time answering the same question

---

## Part 2: Choosing the Right Form

### Form Selection Matrix

| Conversation Type | Deposit Form | Location |
|-------------------|--------------|----------|
| Frequently asked questions | FAQ Document | Project README / Team Wiki |
| Pitfalls encountered | Troubleshooting guide / Guard rules | Project docs/ / lint rules |
| Design decisions | ADR (Architecture Decision Record) | Project docs/adr/ |
| Code review consensus | Team coding standards | Project CONTRIBUTING.md |
| Repetitive operations | Automation scripts | Project scripts/ |
| Thinking patterns | Claude Skills | ~/.claude/skills/ |
| Complete workflows | SOP Documents | Blueprint / Team Wiki |

### Key Distinction: Standards vs Thinking

| Standards | Thinking |
|-----------|----------|
| Tells you "how to do" | Tells you "why think this way" |
| Lists checklists | Explains decision logic |
| Can be executed mechanically | Requires understanding to apply |
| Suitable for lint rules | Suitable for skills |

**Example Comparison**:

```
# Standard (suitable for lint rule)
"Remaining time must use Math.floor for rounding down"

# Thinking (suitable for skill)
"Users perceive time as countdown, showing 3 days means available in 3 days.
Rounding down aligns user expectations with reality, rounding up causes confusion."
```

---

## Part 3: Levels of Abstraction

### Evolution from Specific to Abstract

```
Layer 1: Specific Instance
    ↓
Layer 2: Project Standard
    ↓
Layer 3: General Principle
    ↓
Layer 4: Thinking Pattern
```

**Case Demonstration**:

| Layer | Content |
|-------|---------|
| Specific Instance | "Staking page remaining days uses `Math.floor`" |
| Project Standard | "All countdown displays use `Math.floor`" |
| General Principle | "Time display rounds down, matching countdown psychology" |
| Thinking Pattern | "Before making UI decisions, ask: What is the user thinking?" |

### When to Stop at Which Layer?

- **Specific Instance**: One-time issue, no need to deposit
- **Project Standard**: Reuse within project, add to project docs
- **General Principle**: Cross-project reuse, add to team standards
- **Thinking Pattern**: Guides all decisions, create as skill

---

## Part 4: Deposit Workflow

### Standard Process

```
1. Capture
   ↓ Identify signal, record raw content
2. Analyze
   ↓ Assess value, determine form
3. Abstract
   ↓ From specific to general
4. Store
   ↓ Place in correct location
5. Connect
   ↓ Update index, link related content
6. Validate
   ↓ Actual usage, collect feedback
7. Iterate
   Optimize based on feedback
```

### Quick Deposit Template

When encountering a depositworthy signal, quickly record:

```markdown
## Title: {one-line description}

### Background
- What scenario encountered this?
- Why did it happen?

### Decision/Solution
- How was it solved?
- Why this approach?

### Reusable Points
- When can this be reused?
- What form to abstract into?
```

---

## Part 5: Skill Creation Guide

When conversation content involves **thinking patterns**, it should be deposited as a Claude skill.

### Skill vs Document Difference

| Skill | Document |
|-------|----------|
| Affects Claude behavior | For human reading reference |
| Actively triggers | Passively queried |
| Conveys thinking patterns | Conveys knowledge information |
| Continuously effective | Consulted when needed |

### Skill Creation Principles

1. **Convey thinking, not standards**
   - ❌ "Must use Math.floor"
   - ✅ "Users see countdown, rounding down matches expectations"

2. **Explain why, not how**
   - ❌ "Earnings use green `text-green-500`"
   - ✅ "Green = earning money, user mental model"

3. **Abstract for reusability**
   - ❌ "Staking page remaining days round down"
   - ✅ "Time display rounds down, matches countdown psychology"

4. **Name reflects essence**
   - ❌ `staking-ui-patterns` (too specific)
   - ✅ `user-first-ui` (reflects core thinking)

### Skill Directory Structure

```
~/.claude/skills/
└── skill-name/
    └── SKILL.md      # Only required file
```

---

## Part 6: Case Analysis

### Case: From Staking Page Modifications to user-first-ui Skill

**Background**:
While developing a staking page, the user raised several modification requests:
- Remaining days display uses `Math.floor` not `Math.ceil`
- Earnings-related areas consistently use green
- Principal and earnings withdrawal conditions shown separately
- Data refreshes immediately after successful stake

**Version 1 (Wrong)**:
```markdown
# staking-ui-patterns

- Remaining days use Math.floor
- Earnings use green-50 to emerald-50 gradient
- canWithdraw and canClaimReward judged separately
- Call refetchPoolInfo() after isStakeSuccess
```

Problem: Too specific, only usable for this project.

**Version 2 (Improved)**:
```markdown
# DeFi Staking UI Patterns

## Time Display Standards
- Round down
- Granularity varies with urgency
...
```

Problem: It's a standards checklist, doesn't explain why.

**Version 3 (Correct)**:
```markdown
# User-First UI Design Thinking

Before making UI decisions, ask: **What is the user thinking?**

## Time: User Perspective is Countdown

Users care about "how long remaining", not "how long elapsed".

**Decision**: Round down remaining time
- User sees "3 days remaining" → expects available in 3 days
- Rounding up causes confusion: "It showed 4 days, why is it ready in 3?"
...
```

Key Transformations:
- Name from `staking-ui-patterns` → `user-first-ui`
- Content from "standards checklist" → "thinking pattern + decision reasoning"
- Scope from "staking page" → "all UI design"

---

## Part 7: Deposit Checklist

### During Daily Development

- [ ] After solving a problem: Is this worth recording?
- [ ] When answering questions: How many times have I answered this?
- [ ] During code review: Has this issue been pointed out before?

### When Making Decisions

- [ ] Record decision context (why A not B)
- [ ] Determine if ADR is needed
- [ ] Are related decisions forming a pattern?

### During Project Retrospective

- [ ] What experiences are worth depositing?
- [ ] Are pitfalls documented as guard rules?
- [ ] Are validated workarounds upgraded to formal standards?

### When Creating Skills

- [ ] Conveying thinking patterns or standards checklist?
- [ ] Explained why or just said how?
- [ ] Does name reflect essence or too specific?
- [ ] Is abstraction level sufficient for reuse?

---

## FAQ

### Q: Won't too much depositing cause information overload?

A: The key is layered management:
- High frequency use → Skills (auto-trigger)
- Occasional reference → Documents (search as needed)
- One-time → Don't deposit

### Q: How to determine the right abstraction level?

A: Ask yourself: Can this experience be used in other projects/scenarios?
- Only useful in current project → Project docs
- Useful in similar projects → Team standards
- Useful in all projects → General skill

### Q: What's the difference between skills and documents?

A:
- Skills: Affect Claude behavior, actively effective
- Documents: For human reading, passively queried

---

## References

- [Claude Skills Creation Guide](./006-claude-skill-creation-sop.md)
- [ADR (Architecture Decision Records)](https://adr.github.io/)
- [Team Knowledge Management Best Practices](https://www.notion.so/help/guides/knowledge-management)

---

*Depositing isn't extra work—it's making today's effort continue to generate value tomorrow.*
