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

### Step 1: Copy the Protocol File

Copy `AGENT_PROTOCOL.md` to your project directory.

### Step 2: Send Prompt to AI

Send this prompt to your AI:

```
Read and follow the AGENT_PROTOCOL.md file for this project.
```

### Example

Assume you want AI to help you build a chat application:

```
Read and follow the AGENT_PROTOCOL.md file for this project.

请帮我开发一个类似微信的聊天应用，支持：
- 用户注册登录
- 一对一聊天
- 群组聊天
- 发送图片和文件
```

### What Happens Next

After AI reads the protocol:

1. **First Session (Initialization Agent)**
   - AI detects no `feature_list.json` exists
   - Creates comprehensive feature list with all required features
   - Creates `claude-progress.txt` for progress tracking
   - Creates `init.sh` or `init.bat` for running the project
   - Makes initial git commit
   - Stops (does not start implementation)

2. **Subsequent Sessions (Coding Agent)**
   - AI reads progress file and feature list
   - Selects ONE incomplete feature (highest priority)
   - Implements and tests the feature
   - Commits with clean state
   - Updates progress files
   - Ready for next session

### Why This Works

| Problem | Solution |
|---------|----------|
| AI does too much at once | Protocol enforces one feature per session |
| AI forgets progress between sessions | Progress file provides context continuity |
| AI declares completion too early | Feature list tracks actual completion status |
| AI leaves broken code | Clean state required before commit |
| AI skips testing | E2E testing mandatory before marking feature complete |

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
