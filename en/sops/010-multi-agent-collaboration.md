# Multi-Agent Collaboration System

> Claude as orchestrator, coordinating Gemini/Codex/Claude to answer questions in parallel with multi-perspective insights.

**Document ID**: 010
**Date**: 2025-12-29
**Tags**: `Multi-Agent` `Claude` `Gemini` `Codex` `Collaboration` `Brainstorming`

---

## Overview

This SOP describes how to build a Multi-Agent collaboration system in Claude Code:

- Claude as neutral orchestrator
- Gemini, Codex, and Claude agents answering in parallel
- Structured summary of multiple perspectives
- Support for brainstorming, review, and debate scenarios

## Architecture

```
User Question
    ↓
Claude (Orchestrator)
    ↓
┌───────────────┬───────────────┬───────────────┐
│   Claude      │    Gemini     │    Codex      │
│  (Analyst)    │  (Creative)   │  (Engineer)   │
└───────────────┴───────────────┴───────────────┘
    ↓               ↓               ↓
    └───────────────┴───────────────┘
                    ↓
            Claude (Synthesizer)
                    ↓
            Structured Output
```

## Agent Role Definitions

| Agent | Role | Perspective | Unique Value |
|-------|------|-------------|--------------|
| **Claude** | Analyst | Logical, security-conscious | Risk identification, edge cases |
| **Gemini** | Creative | Divergent thinking, multimodal | Innovative solutions, cross-domain |
| **Codex** | Engineer | Code-oriented, implementation | Technical feasibility, code samples |

## Use Cases

### Case 1: Parallel Q&A

```
/collab {question}
```

Three agents independently answer the same question.

**Use for**: Technical decisions, solution selection, open-ended questions

### Case 2: Brainstorming

```
/brainstorm {topic}
```

Emphasizes creative divergence without judgment.

**Rules**:
- Quantity over quality
- Encourage wild ideas
- Build on others' ideas

### Case 3: Solution Review

```
/review {solution}
```

Multi-angle review to find flaws and risks.

**Role adjustments**:
- Claude → Security reviewer
- Gemini → UX reviewer
- Codex → Technical feasibility reviewer

### Case 4: Debate Mode

```
/debate {topic}
```

Pro/con perspective comparison with neutral arbitration.

**Position assignment**:
- Claude → Pro side
- Gemini → Con side
- Codex → Neutral judge

## Implementation

### Step 1: Parse Question

Identify user intent and determine discussion mode:

```
User: Have three AIs discuss pros and cons of microservices
        ↓
Mode: Debate mode (/debate)
Topic: Pros and cons of microservices
```

### Step 2: Parallel Invocation

Use Task tool to launch three subagents **simultaneously**:

```yaml
# Key: Issue three Task calls in a single message

Task 1 (Claude):
  subagent_type: general-purpose
  prompt: As analyst role, analyze the problem...

Task 2 (Gemini):
  subagent_type: general-purpose
  prompt: Execute gemini "As creative role..." -y

Task 3 (Codex):
  subagent_type: general-purpose
  prompt: Execute codex "As engineer role..."
```

### Step 3: Synthesize Output

Collect results from all three agents, present in structured format:

```markdown
## Question: {user question}

### 🔍 Claude (Analyst)
{response content}

### 💡 Gemini (Creative)
{response content}

### 🔧 Codex (Engineer)
{response content}

## 📊 Summary
| Perspective | Key Point |
|-------------|-----------|
| ... | ... |

## ✅ Consensus
- ...

## ⚠️ Disagreements
- ...

## 🎯 Recommended Actions
1. ...
```

## Cost Control

Multiple agents in parallel = multiple API call costs.

| Mode | Agent Count | Cost | Use Case |
|------|-------------|------|----------|
| Quick | 1 (Claude) | 1x | Simple questions |
| Standard | 2 (Claude + Gemini) | 2x | General discussion |
| Deep | 3 (all) | 3x | Important decisions |

**Default**: Standard mode

**Specify with**:
- "Just Claude" → Quick mode
- "All three" / "Deep analysis" → Deep mode

## Output Format

### Token Limits

Each agent response limited to 500 tokens to avoid information overload.

### Structured Template

```markdown
## Question: {original question}

---

### 🔍 Claude (Analyst)

**Core View**: ...

**Key Considerations**:
- ...
- ...

---

### 💡 Gemini (Creative)

**Core View**: ...

**Innovative Ideas**:
- ...
- ...

---

### 🔧 Codex (Engineer)

**Core View**: ...

**Implementation Suggestions**:
```code
// Sample code
```

---

## 📊 Summary Analysis

### Consensus
...

### Disagreements
...

### Recommendations
...
```

## Error Handling

| Situation | Handling |
|-----------|----------|
| Single agent timeout (>60s) | Skip agent, mark as "No Response" |
| Single agent error | Show error, continue with others |
| All agents fail | Fallback to Claude-only response |

## Important Notes

1. **Parallel Execution**: All three Tasks must be in the same message
2. **Sensitive Info**: Don't pass keys/passwords to Gemini/Codex
3. **Context Isolation**: Each agent runs in isolated subagent
4. **Cost Control**: Use deep mode only for important questions

## Skill Dependencies

Ensure these skills are enabled:

- `gemini-cli` - Gemini CLI multimodal assistant
- `codex-cli` - Codex CLI programming assistant

## Example Conversation

**User**:
```
/collab I want to build an AI-powered code review tool, any suggestions?
```

**Output**:
```
## Question: AI-powered code review tool suggestions

---

### 🔍 Claude (Analyst)

Core challenge is **accuracy vs false positive rate balance**.

Key considerations:
- Multi-language support needed
- Enterprise needs on-premise deployment
- Integration with existing CI/CD

Potential risks:
- LLM may generate incorrect fixes
- Performance issues with large codebases

---

### 💡 Gemini (Creative)

Could introduce "Code Smell Map" concept for visualization.

Innovative ideas:
- Combine Git history to analyze bug hotspots
- Use knowledge graphs to link similar historical fixes
- Support voice/screenshot input for issue descriptions

---

### 🔧 Codex (Engineer)

Tech stack suggestion: tree-sitter + LLM

Implementation architecture:
```python
class CodeReviewer:
    def __init__(self):
        self.parser = TreeSitterParser()
        self.llm = LLMClient()

    def review(self, code: str) -> List[Issue]:
        ast = self.parser.parse(code)
        context = self.extract_context(ast)
        return self.llm.analyze(context)
```

---

## 📊 Summary

### ✅ Consensus
- Multi-language support needed
- CI/CD integration required
- LLM output needs human confirmation

### ⚠️ Disagreements
- Architecture: Claude suggests caution, Gemini suggests bold innovation

### 🎯 Recommendations
1. Start with single-language MVP
2. Use tree-sitter for basic parsing
3. Design human confirmation workflow
```

## Multi-Round Iterative Discussion

Supports multi-round interaction between agents for responding, questioning, and supplementing.

### Trigger Methods

```
/collab --rounds=3 {question}     # Specify 3 rounds
/debate --deep {topic}            # Auto-enable multi-round
```

### Workflow

```
Round 1: Independent answers
    ↓
(Summary compression)
    ↓
Round 2: Respond/rebut after seeing others' views
    ↓
(Summary compression)
    ↓
Round 3: Reach consensus or clarify disagreements
    ↓
Final summary
```

### Context Passing (Summary Compression)

After each round, Claude orchestrator summarizes results; next round only receives summary:

```
Round N+1 prompt template:

【History Summary】
Round 1 key points:
- Analyst: {Claude core view, 50 chars}
- Creative: {Gemini core view, 50 chars}
- Engineer: {Codex core view, 50 chars}

Round 2 key points:
- ...

【This Round Task】
Please supplement or rebut the above views...

Question: {original question}
```

### Summary Compression Implementation

After collecting each round's results, call Claude to generate summary:

```yaml
Task: Generate round summary
  subagent_type: general-purpose
  model: haiku  # Use cheap model for summaries
  prompt: |
    Compress each of the following three views into one sentence (max 50 chars):

    Analyst original: {claude_response}
    Creative original: {gemini_response}
    Engineer original: {codex_response}

    Output format:
    - Analyst: ...
    - Creative: ...
    - Engineer: ...
```

### Multi-Round Output Format

```markdown
## Question: {user question}

---

### Round 1: Independent Answers

#### 🔍 Claude (Analyst)
{detailed answer}

#### 💡 Gemini (Creative)
{detailed answer}

#### 🔧 Codex (Engineer)
{detailed answer}

---

### Round 2: Interactive Discussion

#### 🔍 Claude (Analyst)
> Regarding Gemini's view: ...
{supplement/rebuttal}

#### 💡 Gemini (Creative)
> Regarding Codex's view: ...
{supplement/rebuttal}

#### 🔧 Codex (Engineer)
> Regarding Claude's view: ...
{supplement/rebuttal}

---

### Round 3: Reaching Consensus

{Claude orchestrator summary}

---

## 📊 Final Summary

### ✅ Consensus Points
- ...

### ⚠️ Remaining Disagreements
- ...

### 🎯 Recommended Actions
1. ...
```

### Round Control Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `--rounds=N` | Number of discussion rounds | 1 |
| `--max-rounds` | Maximum round limit | 5 |
| `--auto-stop` | Auto-stop when consensus reached | true |

## References

- [Multi-Agent Systems Overview](https://en.wikipedia.org/wiki/Multi-agent_system)
- [Claude Code Skills Documentation](https://docs.anthropic.com/claude-code/skills)
- [Gemini CLI Documentation](https://github.com/google-gemini/gemini-cli)
- [Codex CLI Documentation](https://github.com/openai/codex)
