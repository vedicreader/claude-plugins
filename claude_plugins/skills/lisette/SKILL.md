---
name: lisette
description: Call LLMs using Lisette, a simplified wrapper around LiteLLM supporting 100+ providers. Use when making LLM API calls, building chat applications, using tool calling, or working with Claude and other models.
user-invocable: true
allowed-tools: Bash, Read
---

# Lisette — Simplified LLM Wrapper

Lisette wraps LiteLLM to make working with 100+ LLM providers simple. It provides a stateful `Chat` class with tool calling, streaming, and async support.

## Core: `Chat` Class

```python
from lisette import Chat

# Create a chat session (stateful — remembers history)
chat = Chat(model='claude-sonnet-4-6')

# Simple call
response = chat('What is 2+2?')
print(response)  # '4'

# Conversation continues with history
response = chat('Now multiply that by 10')
print(response)  # '40'
```

## Model IDs

```python
# Anthropic Claude models
chat = Chat('claude-opus-4-6')      # Most capable
chat = Chat('claude-sonnet-4-6')    # Fast and capable (default for most tasks)
chat = Chat('claude-haiku-4-5-20251001')  # Fastest

# OpenAI
chat = Chat('gpt-4o')
chat = Chat('gpt-4o-mini')

# Gemini
chat = Chat('gemini/gemini-2.0-flash')

# Local (Ollama)
chat = Chat('ollama/llama3')
```

## System Prompt

```python
chat = Chat(
    model='claude-sonnet-4-6',
    sp='You are a helpful Python expert. Always include type hints.'
)
```

## Tool Calling

```python
from lisette import Chat

def get_weather(city: str) -> str:
    """Get current weather for a city."""
    return f'Sunny, 22°C in {city}'

chat = Chat('claude-sonnet-4-6', tools=[get_weather])
response = chat('What is the weather in London?')
# Lisette automatically calls get_weather('London') and returns result
```

## Streaming

```python
chat = Chat('claude-sonnet-4-6', stream=True)
for chunk in chat('Tell me a story'):
    print(chunk, end='', flush=True)
```

## Async

```python
import asyncio
from lisette import Chat

chat = Chat('claude-sonnet-4-6')

async def main():
    response = await chat.async_call('Hello!')
    print(response)

asyncio.run(main())
```

## Web Search Integration

```python
chat = Chat('claude-sonnet-4-6', web_search=True)
response = chat('What happened in the news today?')
```

## Prompt Caching (Claude)

```python
chat = Chat('claude-sonnet-4-6', cache=True)
# Repeated long system prompts are cached automatically
```

## Accessing History

```python
chat = Chat('claude-sonnet-4-6')
chat('Hello')
chat('How are you?')

print(chat.h)  # Full message history as list of dicts
```

## Multiple Turns with Context

```python
chat = Chat('claude-sonnet-4-6', sp='You are a code reviewer.')

code = open('myfile.py').read()
issues = chat(f'Review this code:\n```python\n{code}\n```')
fix = chat('Now fix the main issue you found')
```

## Vision / Multimodal

```python
from lisette import Chat, image_from_path

chat = Chat('claude-sonnet-4-6')
img = image_from_path('screenshot.png')
response = chat(['Describe this image', img])
```

## Installation

```bash
pip install lisette
```

Set API keys:
```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
```

Repo: https://github.com/answerdotai/Lisette
