# AI Long-Running Agent Protocol

Follow this protocol when working across multiple context windows.

---

## First Session: Initialization Agent

If this is the **first session** (no `feature_list.json` exists):

1. **Analyze the task** and create `feature_list.json` with ALL features needed:
```json
{
  "features": [
    {
      "id": "F001",
      "category": "core|ui|api|etc",
      "description": "What this feature does",
      "steps": ["Step 1", "Step 2", "Step 3"],
      "priority": "high|medium|low",
      "passes": false
    }
  ]
}
```

2. **Create `init.sh`** (or `init.bat` on Windows) to start the development server

3. **Create `claude-progress.txt`** with initial state:
```
# Project Progress
## Status: INITIALIZING
## Last Updated: [timestamp]
## Completed Features: 0 / [total]
## Current Focus: Setup
## Notes: Initial setup complete
```

4. **Make initial git commit** with message: `"Initial project setup with feature list"`

5. **Stop here** - Let the next session begin actual implementation

---

## Subsequent Sessions: Coding Agent

Every session MUST follow this sequence:

### Step 1: Get Bearings
```
- Run pwd to confirm working directory
- Read claude-progress.txt
- Read feature_list.json
- Run git log --oneline -10
```

### Step 2: Verify Environment
```
- Run init.sh/init.bat to start dev server
- Perform basic end-to-end tests
- Fix any bugs BEFORE starting new work
```

### Step 3: Pick ONE Feature
- Select the highest priority incomplete feature from feature_list.json
- Work on ONLY that feature this session

### Step 4: Implement & Test
- Implement the feature
- **MANDATORY**: Run actual end-to-end tests (not just code review)
- Use browser automation or real API calls when applicable
- DO NOT mark feature as complete without verification

### Step 5: Clean State Commit
- Ensure code works (no half-implemented states)
- Commit with descriptive message
- Update `claude-progress.txt`:
```
# Project Progress
## Status: IN_PROGRESS
## Last Updated: [timestamp]
## Completed Features: [n] / [total]
## Current Focus: [feature id]
## Notes: [what was done this session]
## Next Steps: [suggested next feature]
```

### Step 6: Update Feature List
- Only change `passes: true` if feature is verified working
- NEVER delete or modify test steps

---

## Critical Rules

| Rule | Description |
|------|-------------|
| ONE FEATURE | Work on one feature per session |
| CLEAN STATE | Code must be working before session ends |
| TEST EVERYTHING | End-to-end tests required, not just code inspection |
| DOCUMENT PROGRESS | Always update progress file and git |
| NO DELETIONS | Never delete features from list, only mark passes |

---

## Failure Modes to Avoid

| Failure Mode | Prevention |
|--------------|------------|
| Doing too much at once | Stick to one feature |
| Premature completion | Require real tests |
| Leaving broken state | Verify before commit |
| Forgetting progress | Read progress file first |
| Guessing state | Use git log and progress file |

---

## Quick Reference

```
START SESSION:
  pwd → read progress → read features → git log → run init.sh → test basics

END SESSION:
  test feature → commit → update progress → update feature_list.json
```
