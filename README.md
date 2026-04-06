# AI Text Humanizer — Claude Code Sub-Agent Setup Guide

## What This Does

A two-agent feedback loop in Claude Code:

1. **Academic Humanizer** rewrites AI-generated text to sound human.
2. **AI Text Detector** scores the output and flags AI patterns.
3. The detector's feedback goes back to the humanizer. Repeat until the text passes (`ai_probability < 20`).

```
AI Text → Humanizer → Detector → Pass? → Yes → Done
                ↑                    │
                └──── No (feedback) ─┘
```

---

## Setup

### 1. Create project structure

```bash
mkdir ai-text-humanizer && cd ai-text-humanizer
mkdir -p .claude/agents
```

### 2. Add the agent files

Copy the two prompt files from the repo into `.claude/agents/`:

- [academic-humanizer.md](https://github.com/elitex45/ai-text-humanizer/blob/main/academic-humanizer.md) → `.claude/agents/academic-humanizer.md`
- [ai-text-detector.md](https://github.com/elitex45/ai-text-humanizer/blob/main/ai-text-detector.md) → `.claude/agents/ai-text-detector.md`

Each file needs YAML frontmatter added at the top. Wrap the existing prompt content below it.

**`.claude/agents/academic-humanizer.md`** — add this frontmatter before the existing prompt:

```yaml
---
name: academic-humanizer
description: Rewrites AI-generated academic text to sound naturally human-written. Use when humanizing AI text.
model: sonnet
---
```

**`.claude/agents/ai-text-detector.md`** — add this frontmatter before the existing prompt:

```yaml
---
name: ai-text-detector
description: Analyzes text for AI-generated patterns and returns a JSON report with flagged sections and suggestions. Use when checking if text sounds AI-generated.
model: sonnet
---
```

### 3. Verify agents are loaded

```bash
claude agents
```

You should see both `academic-humanizer` and `ai-text-detector` listed.

---

## Usage

### Start Claude Code

```bash
cd ai-text-humanizer
claude
```

### Run the loop

Paste your AI-generated text and tell Claude to run the workflow:

```
Here is my AI-generated text:

"""
<paste your text here>
"""

Use the academic-humanizer agent to rewrite this text.
Then use the ai-text-detector agent to analyze the result.
If ai_probability is 20 or above, feed the detector's flagged_sections
back to the academic-humanizer and repeat until it passes.
Output the final humanized text when it passes.
```

Claude will chain the two sub-agents automatically, passing detector feedback back to the humanizer each round.

---

## How It Works Internally

- **Sub-agents** run in their own context window with the custom system prompt from each `.md` file.
- Claude acts as the **orchestrator** — it reads the detector's JSON output, checks the `pass` field, and decides whether to loop again or return the final text.
- Each agent only sees what Claude passes to it, keeping context clean.

For more on Claude Code sub-agents, see the [official docs](https://code.claude.com/docs/en/sub-agents).

---

## Reference

| File | Purpose |
|------|---------|
| [`academic-humanizer.md`](https://github.com/elitex45/ai-text-humanizer/blob/main/academic-humanizer.md) | System prompt for the humanizer agent |
| [`ai-text-detector.md`](https://github.com/elitex45/ai-text-humanizer/blob/main/ai-text-detector.md) | System prompt for the detector agent |
