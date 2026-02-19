# AI Long-Running Agent Protocol

A protocol for AI agents working across multiple context windows. Inspired by effective software engineering practices.

---

## The Core Problem

AI agents working across multiple context windows face two critical failure modes:

1. **Doing too much at once** - Agents try to complete everything in one session, leaving half-implemented features and undocumented progress
2. **Premature completion** - Agents declare project complete before proper verification

---

## Dual-Agent Solution

| Agent Type | Role |
|------------|------|
| **Initialization Agent** | First session: sets up environment, creates feature list, makes initial commit |
| **Coding Agent** | Subsequent sessions: works on ONE feature, leaves clean state for next session |

---

## Phase 1: Initialization Agent

If `feature_list.json` does NOT exist, you are the **Initialization Agent**.

### Your Tasks

1. **Analyze the prompt** and create a comprehensive `feature_list.json`:
```json
{
  "project": "Project Name",
  "total_features": 0,
  "features": [
    {
      "id": "F001",
      "category": "functional|ui|api|auth|etc",
      "description": "Detailed description of what this feature does",
      "steps": [
        "Step 1: Specific action to verify",
        "Step 2: Another verification step",
        "Step 3: Expected outcome"
      ],
      "priority": "high|medium|low",
      "passes": false
    }
  ]
}
```

**Important**: All features must start with `passes: false`. This file uses JSON because it's harder for models to accidentally corrupt compared to Markdown.

2. **Create `init.sh`** (or `init.bat` on Windows):
   - Script to start the development server
   - Include any necessary setup commands

3. **Create `claude-progress.txt`**:
```
# Project Progress
## Status: INITIALIZING
## Last Updated: YYYY-MM-DD HH:MM
## Completed Features: 0 / [total]
## Current Focus: Setup
## Last Commit: [commit hash]
## Notes: Initial environment setup complete
## Next Steps: Begin feature implementation
```

4. **Make initial git commit**:
```
git add .
git commit -m "Initial project setup with feature list"
```

5. **STOP** - Do NOT begin implementation. Let the next session start fresh.

---

## Phase 2: Coding Agent

If `feature_list.json` EXISTS, you are the **Coding Agent**.

### Session Start Sequence

Follow this EXACT sequence at the start of EVERY session:

```
1. pwd                    # Confirm working directory
2. Read claude-progress.txt
3. Read feature_list.json
4. git log --oneline -20  # Understand recent work
5. Run init.sh/init.bat   # Start development server
6. Test basic functionality # Verify environment is working
7. Fix any bugs found BEFORE starting new work
```

### Session Work Flow

#### Step 1: Select ONE Feature
- Choose the highest priority incomplete feature (`passes: false`)
- Work on ONLY this feature for the entire session

#### Step 2: Implement
- Write the code for the selected feature
- Keep changes focused and minimal

#### Step 3: Verify with End-to-End Tests
**MANDATORY**: You MUST verify the feature works end-to-end:
- Use browser automation tools (Puppeteer, Playwright) for web apps
- Use real API calls for backend features
- Run actual user-like interactions, NOT just code review
- DO NOT rely solely on `curl` or unit tests

#### Step 4: Clean State Commit
Before ending the session, ensure:
- Code is working (no broken states)
- No half-implemented features
- Code is documented and organized
- Suitable for merging to main branch

Then:
```
git add .
git commit -m "[Feature ID] Description of what was implemented"
```

#### Step 5: Update Progress Files

Update `claude-progress.txt`:
```
# Project Progress
## Status: IN_PROGRESS
## Last Updated: YYYY-MM-DD HH:MM
## Completed Features: [n] / [total]
## Current Focus: [feature id that was just completed]
## Last Commit: [commit hash]
## Notes: [what was done this session]
## Next Steps: [suggested next feature to work on]
```

Update `feature_list.json`:
- **ONLY change `passes: true`** for the completed feature
- **NEVER delete or modify** feature descriptions or test steps

---

## Critical Rules

### Forbidden Actions

| Forbidden | Reason |
|-----------|--------|
| Delete features from list | Causes feature loss and bugs |
| Modify test steps | Corrupts verification criteria |
| Work on multiple features | Leads to half-implemented states |
| Skip end-to-end testing | Misses real-world bugs |
| Leave broken code | Next session wastes time fixing |

### Required Actions

| Required | Reason |
|----------|--------|
| Read progress file first | Understand current state |
| Test before starting new work | Catch bugs early |
| One feature per session | Maintain focus and quality |
| End-to-end verification | Ensure feature actually works |
| Commit with clean state | Enable easy rollback |
| Update both progress files | Context continuity |

---

## Failure Modes & Solutions

| Problem | Initialization Agent | Coding Agent |
|---------|---------------------|--------------|
| Premature project completion | Create comprehensive feature list with all features | Read feature list first; select incomplete feature |
| Leaving broken/undocumented state | Create initial git repo and progress file | Read progress + git log; test basics; commit with updates |
| Marking features complete without testing | N/A | Verify with E2E tests before marking `passes: true` |
| Wasting time figuring out how to run app | Write `init.sh` script | Read `init.sh` and run it |

---

## Definition of Clean State

A "clean state" means code that is:
- Working without major bugs
- Properly organized and documented
- Suitable for merging to main branch
- Ready for next developer to start new feature without cleanup

---

## Quick Reference Card

```
╔══════════════════════════════════════════════════════════════╗
║                    SESSION START                              ║
║  1. pwd                                                       ║
║  2. Read claude-progress.txt                                  ║
║  3. Read feature_list.json                                    ║
║  4. git log --oneline -20                                     ║
║  5. Run init.sh/init.bat                                      ║
║  6. Test basic functionality                                  ║
║  7. Fix bugs BEFORE new work                                  ║
║  8. Pick ONE feature (highest priority, passes: false)        ║
║  9. Implement feature                                         ║
║  10. E2E test verification                                    ║
║  11. git commit (clean state)                                 ║
║  12. Update claude-progress.txt                               ║
║  13. Update feature_list.json (passes: true only)             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Example Session

```
[Assistant] I'll start by getting my bearings and understanding the current state.

[Tool Use] <bash - pwd>

[Tool Use] <read - claude-progress.txt>

[Tool Use] <read - feature_list.json>

[Assistant] Let me check the git log to see recent work.

[Tool Use] <bash - git log --oneline -20>

[Assistant] Now let me check if there's an init.sh script to restart the servers.

[Tool Use] <bash - ./init.sh>

[Assistant] Excellent! Now let me verify that fundamental features are still working.

<Tests basic functionality>

[Assistant] Based on verification, the core features are working correctly. 
Let me review the feature list to find the next priority feature to implement.

<Selects F005 from feature_list.json>

<Implements feature F005>

<Runs E2E tests to verify>

[Assistant] Feature F005 is working. Let me commit and update progress.

[Tool Use] <bash - git add . && git commit -m "F005: Add user authentication">

[Tool Use] <write - claude-progress.txt>

[Tool Use] <write - feature_list.json - update passes: true for F005>

[Assistant] Session complete. F005 implemented and verified. 
Next suggested feature: F006 (User profile page).
```

---

## Credit

Based on Anthropic's research on effective constraints for long-running AI agents.
