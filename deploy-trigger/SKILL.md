---
name: deploy-trigger
description: Trigger deployment/builds, fetch logs, stream progress, and run HTTP health checks
argument-hint: [Application/Service resource identifier]
---

# /deploy-trigger — Trigger Deployment & Monitor Logs

You are executing the `/deploy-trigger` skill. Your task is to start the deployment, stream build and startup logs, and perform domain health checks to confirm successful rollout.

## Ethos

!`sh .adlc/partials/ethos-include.sh 2>/dev/null || sh ~/.claude/skills/partials/ethos-include.sh`

## Input

Resource ID / Domain: $ARGUMENTS

## Instructions

### Step 1: Trigger Build/Deploy
1. Send deployment trigger signal via API, SSH commands (`docker compose up --build -d`), or Git push to the target platform.
2. Store the build ID or deployment session token.

### Step 2: Stream & Capture Logs
1. Fetch and print the real-time build logs to show progress (compiling, installing dependencies, copying files).
2. If the build succeeds, monitor the container runtime logs (database connections, startup messages, listen port binding).
3. Save all streamed build and startup logs to `.adlc/logs/deploy-latest.log` (overwriting the previous log) and `.adlc/logs/deploy-[timestamp].log` (where `timestamp` is the execution time, e.g., `20260520-171500`, for audit history).
4. If an error is detected in build/runtime logs, ensure the failure details are written to the logs, and exit with code 1, outputting the error snippet clearly.

### Step 3: Health Check
1. Once logs show successful binding:
   - Perform HTTP health checks (sending GET requests to the application's domain).
   - If the request returns status code 2xx or 3xx, consider the deployment successful.
   - If Nginx returns 502 Bad Gateway or the connection times out, exit with code 1.

### Step 4: Summary Output
Output the status of the deployment:
- **Deployment Status:** Success / Failed
- **Build Duration:** e.g., 45s
- **Health Check Response:** e.g., HTTP 200 OK
