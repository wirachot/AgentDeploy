---
name: deploy-heal
description: Parse failed build/runtime logs, diagnose root cause, auto-apply code or config fixes, commit and push changes
argument-hint: [Error details or log file paths]
---

# /deploy-heal — AI DevOps Self-Healing Agent

You are executing the `/deploy-heal` skill. Your task is to act as a self-healing agent to parse error logs, identify why a deployment failed, automatically modify files/configs to address the issue, and commit/push changes.

## Ethos

!`sh .adlc/partials/ethos-include.sh 2>/dev/null || sh ~/.claude/skills/partials/ethos-include.sh`

## Input

Error Logs/Failure Reason: $ARGUMENTS (Can be raw logs, error summary, or a path to a log file such as `.adlc/logs/deploy-latest.log`. If no argument is provided, default to reading `.adlc/logs/deploy-latest.log`).

## Instructions

### Step 1: Parse logs & Diagnose
1. Inspect the provided build or runtime logs. If a path to a log file is given (or if no argument was passed and `.adlc/logs/deploy-latest.log` exists), read the file's content directly to obtain the logs.
2. Formulate a diagnostic assessment identifying:
   - **Error Category:** (e.g. Missing Dependency, Port Conflict, Database Connection Failure, Missing Environment Variable, Syntactic Bug).
   - **Target File & Line:** Where the error occurred.
   - **Root Cause Explanation:** What needs to be changed.

### Step 2: Apply Auto-Correction
1. Use `replace_file_content` or `multi_replace_file_content` to fix the codebase or configuration. Examples:
   - Fix port conflict: Change port bindings in code or `Dockerfile`.
   - Missing dependency: Add the missing package to `package.json` or `requirements.txt`.
   - Env mismatch: Update configurations to match expected env variable names.
   - DB Connection URL: Correct the connection parsing code.
2. Verify locally that files are successfully edited.

### Step 3: Git Commit & Push
1. Check the local git status: `git status`.
2. Commit the auto-healed changes:
   ```bash
   git add <modified-files>
   git commit -m "chore(deploy): auto-heal [error-type] - fix build/runtime crash [REQ-xxx]"
   ```
3. Push the commits to the remote repository so the deployment platform receives the update:
   ```bash
   git push origin <branch-name>
   ```

### Step 4: Summary Output
Output the self-healing summary:
- **Diagnosed Error:**
- **Files Modified:**
- **Commit SHA/Message:**
- **Status:** Pushed & ready for redeployment
