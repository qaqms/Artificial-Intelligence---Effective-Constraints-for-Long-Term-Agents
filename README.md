# AI Long-Running Agent Protocol

A structured protocol for AI agents working across multiple context windows. Inspired by effective software engineering practices.

## Problem

AI agents working across multiple context windows face two critical failure modes:

1. **Doing too much at once** - Agents try to complete everything in one session, leaving half-implemented features
2. **Premature completion** - Agents declare victory before proper verification

## Solution

A dual-agent approach:

| Agent | Role |
|-------|------|
| **Initialization Agent** | Sets up environment, creates feature list, makes initial commit |
| **Coding Agent** | Works on one feature per session, leaves clean state |

## Quick Start

Include `AGENT_PROTOCOL.md` in your project and instruct the AI:

```
Read and follow the AGENT_PROTOCOL.md file for this project.
```

## Key Components

### feature_list.json
Complete feature inventory with pass/fail status:
```json
{
  "features": [
    {
      "id": "F001",
      "description": "Feature description",
      "steps": ["Step 1", "Step 2"],
      "priority": "high",
      "passes": false
    }
  ]
}
```

### claude-progress.txt
Session-by-session progress log for context continuity.

### init.sh / init.bat
Development server startup script.

## Workflow

```
Session Start
  ↓ pwd → read progress → read features → git log → init
  ↓ Test basic functionality
  ↓ Pick ONE feature
  ↓ Implement & verify with E2E tests
  ↓ Commit with descriptive message
  ↓ Update progress file
Session End
```

## Core Rules

| Rule | Description |
|------|-------------|
| One Feature | Work on one feature per session |
| Clean State | Code must work before session ends |
| Test Everything | End-to-end tests required |
| Document Progress | Update progress file and git |
| No Deletions | Never delete features from list |

## Credit

Based on [Anthropic's research](https://www.anthropic.com/research) on long-running AI agents.

## License

MIT License
