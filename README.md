# AI Long-Running Agent Protocol

A comprehensive protocol for AI agents working across multiple context windows. Inspired by Anthropic's research on effective constraints for long-running agents.

## Problem Solved

AI agents working across multiple context windows face two critical failure modes:

1. **Doing too much at once** - Leaving half-implemented features and undocumented progress
2. **Premature completion** - Declaring project complete before proper verification

## Solution

A dual-agent approach:

| Agent | Role |
|-------|------|
| **Initialization Agent** | Sets up environment, creates feature list, makes initial commit |
| **Coding Agent** | Works on ONE feature per session, leaves clean state |

## Quick Start

Include `AGENT_PROTOCOL.md` in your project:

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
      "category": "functional",
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
    ↓
pwd → read progress → read features → git log → init.sh
    ↓
Test basic functionality → Fix bugs
    ↓
Pick ONE feature (highest priority, passes: false)
    ↓
Implement → E2E test verification
    ↓
Commit (clean state) → Update progress files
    ↓
Session End
```

## Core Rules

| Rule | Description |
|------|-------------|
| One Feature | Work on one feature per session |
| Clean State | Code must work before session ends |
| E2E Testing | End-to-end tests required, not just code review |
| Document Progress | Update progress file and git |
| No Deletions | Never delete features or modify test steps |

## Files

| File | Description |
|------|-------------|
| `AGENT_PROTOCOL.md` | Core protocol documentation |
| `README.md` | Project overview |
| `LICENSE` | MIT License |

## License

MIT License - see [LICENSE](LICENSE) file.
