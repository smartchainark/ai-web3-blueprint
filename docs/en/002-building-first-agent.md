# Building Your First AI Agent

> A quick-start guide to creating a functional AI agent with tool use.

**Document**: 002
**Date**: 2024-12-28
**Tags**: `agent` `tutorial` `beginner`

---

## Introduction

In this tutorial, we'll build a simple but functional AI agent that can:
- Answer questions using web search
- Perform calculations
- Read and summarize files

## Prerequisites

- Python 3.10+
- OpenAI API key (or Anthropic)
- Basic Python knowledge

## Step 1: Setup

```bash
pip install openai requests
```

```python
import openai
import json

client = openai.OpenAI()
```

## Step 2: Define Tools

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Perform mathematical calculations",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string", "description": "Math expression"}
                },
                "required": ["expression"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the web for information",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"}
                },
                "required": ["query"]
            }
        }
    }
]
```

## Step 3: Implement Tool Functions

```python
def calculate(expression: str) -> str:
    try:
        result = eval(expression)  # Note: Use safer eval in production
        return str(result)
    except Exception as e:
        return f"Error: {e}"

def search_web(query: str) -> str:
    # Simplified - use a real search API in production
    return f"Search results for: {query}"

tool_functions = {
    "calculate": calculate,
    "search_web": search_web
}
```

## Step 4: Agent Loop

```python
def run_agent(user_message: str):
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.chat.completions.create(
            model="gpt-4",
            messages=messages,
            tools=tools
        )

        message = response.choices[0].message

        if message.tool_calls:
            for tool_call in message.tool_calls:
                func = tool_functions[tool_call.function.name]
                args = json.loads(tool_call.function.arguments)
                result = func(**args)

                messages.append(message)
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": result
                })
        else:
            return message.content
```

## Step 5: Run It

```python
result = run_agent("What is 42 * 17 and what's the latest news about AI?")
print(result)
```

## What's Next?

- Add more tools (file reading, API calls)
- Implement memory for context persistence
- Add error handling and retries

## Summary

You've built a basic AI agent! While simple, this pattern scales to complex applications.

---

**References**:
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use](https://docs.anthropic.com/claude/docs/tool-use)
