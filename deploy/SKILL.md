---
name: deploy
description: Orchestrate end-to-end auto-deployment by sequentially running codebase analysis, env configuration, provisioning, build trigger, and self-healing.
argument-hint: [Optional target domain, VPS address, or Git repo URL]
---

# /deploy — DevOps & Auto-Deployment Pipeline

You are the autonomous DevOps orchestrator. Your task is to guide the application through codebase analysis, environment configuration, resource provisioning, deployment, and self-healing by sequentially invoking sub-skills.

## Ethos

!`sh .adlc/partials/ethos-include.sh 2>/dev/null || sh ~/.claude/skills/partials/ethos-include.sh`

## Context

- Deployment config: !`cat .adlc/config.yml 2>/dev/null || echo "No deployment config found"`
- Active spec: !`grep -rl 'status: in-progress' .adlc/specs/*/requirement.md 2>/dev/null | head -1 || echo "No active spec"`

## Input

Target/Arguments: $ARGUMENTS

## Autonomous Orchestration Pipeline

Execute the following phases in sequence. Each phase delegates work to its corresponding sub-skill. Maintain a local deployment state to track progress.

---

### Phase 0: Preflight & Resolve Target
1. Resolve target provider details and credentials from `$ARGUMENTS` and `.adlc/config.yml`.
2. Verify API tokens are present (e.g., `COOLIFY_API_KEY` for Coolify API, or SSH credentials for VPS).
3. Log status: "Preflight verified. Initializing deployment pipeline."

---

### Phase 1: Codebase Analysis
**Action:** Invoke `/deploy-analyze` with the target codebase/repository path.
1. Capture output details (identified stack, listening ports, database dependencies).
2. Verify that Dockerfile, docker-compose.yml, or nixpacks.toml configurations are generated/validated.
3. Log status: "Codebase analysis completed. Scaffolded configurations verified."

---

### Phase 2: Environment Configuration
**Action:** Invoke `/deploy-env` with the target codebase path.
1. Auto-discover environment variables.
2. Interactively gather required credentials/secrets from the user or configure secure defaults.
3. Keep the resolved key-value configuration ready for provisioning.
4. Log status: "Environment variables configuration ready."

---

### Phase 3: Service Provisioning
**Action:** Invoke `/deploy-provision` with the target details, stack data, and env configuration.
1. Call Coolify API or VPS shell commands to check for/create required database services.
2. Register the application, exposing the correct container port and mapping the public domain.
3. Inject the complete environment variables into the application settings.
4. Log status: "Resources provisioned. Target platform configured."

---

### Phase 4: Trigger & Build Monitor
**Action:** Invoke `/deploy-trigger` with the provisioned application identifier.
1. Initiate the build/deployment on the target platform.
2. Stream build logs in real-time.
3. Monitor container startup logs to ensure port binding succeeds.
4. Save the full build/startup output to `.adlc/logs/deploy-latest.log` (overwrites existing) and `.adlc/logs/deploy-[timestamp].log` (for history).
5. If deployment succeeds (health check HTTP 200 OK), skip to Phase 6.
6. If deployment fails (build crash, Nginx 502, timeout), ensure logs are fully captured in `.adlc/logs/deploy-latest.log` and proceed to Phase 5.

---

### Phase 5: Self-Healing & Redeployment Loop
**Action:** Invoke `/deploy-heal` with the path to the error log file `.adlc/logs/deploy-latest.log`.
1. Let the healing agent diagnose the failure cause (missing dependency, wrong port, DB connection error) by reading the deploy log.
2. Auto-edit files in the codebase (code or config) to resolve the error.
3. Commit the changes and push to the remote repository.
4. **Redeploy Loop:** Return to Phase 4 to re-trigger build and monitor.
5. **Iteration Limit:** Allow up to **3 healing iterations**. If the deployment still fails after 3 tries, halt and present the parsed logs and recommended fixes to the user.

---

### Phase 6: Health Verification & Handover
1. Perform a final HTTP GET verification request to the application domain.
2. Display a summary report:
   - **Deployment Status:** Active / Online
   - **Domain URL:** [Clickable Link]
   - **Exposed Port:** e.g., 3000
   - **Database Services:** Connected
   - **Auto-healed Issues:** Details of any fixes applied in Phase 5.
3. Log status: "Deployment successful. Handover complete."

## Error Handling

- **Authentication failures:** Halt in Phase 0 and request credentials.
- **Scaffolding failures:** If configurations cannot be scaffolded in Phase 1, ask the user.
- **Redeployment limit reached:** If Phase 5 hits the 3-try threshold, halt and request human guidance.
