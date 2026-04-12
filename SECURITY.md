# Security Policy — Sentinel Fork of Hermes Agent

This is the Project Sentinel fork of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent).

## Reporting Vulnerabilities

**For vulnerabilities in the upstream Hermes Agent project**, report to the NousResearch team via their security channels.

**For vulnerabilities specific to this fork's modifications**, report privately via GitHub Security Advisories on this repository, or contact the maintainer directly.

Do not open public issues for security vulnerabilities.

## Fork Security Modifications

This fork applies the following security hardening over the upstream Hermes Agent:

### 1. Docker Approval Bypass Removed (`tools/approval.py`)

**Upstream behavior:** All dangerous commands are auto-approved when running inside Docker, Singularity, Modal, or Daytona containers.

**Why this is unsafe for Sentinel:** Our containers have persistent bind mounts to secrets, Obsidian vault, workspace files, and project repositories. Auto-approving destructive commands (e.g., `rm -rf` on mounted directories) risks data loss.

**Fix:** Removed the `env_type in ("docker", ...)` early-return bypass in both `check_dangerous_command()` and `check_all_command_guards()`.

### 2. approvals.mode="off" Bypass Removed (`tools/approval.py`)

**Upstream behavior:** Setting `approvals.mode: "off"` in `config.yaml` disables all approval and security checks globally.

**Why this is unsafe:** A config path that disables all safety checks is a single point of failure. Config must never collapse all safety.

**Fix:** Removed the `approval_mode == "off"` bypass in `check_all_command_guards()`.

### 3. YOLO Mode (Retained — Opt-In Only)

**Upstream behavior:** `HERMES_YOLO_MODE` env var or session-scoped `/yolo` bypasses approvals.

**Sentinel decision (2026-04-12):** YOLO mode is **retained** as a deliberate opt-in. The session-scoped `/yolo` (from upstream PR) is the preferred form — it limits blast radius to the active session rather than the entire agent runtime. Process-scoped `HERMES_YOLO_MODE` env var remains available for local CLI use.

**Operational policy:**
- `HERMES_YOLO_MODE` must never appear in any deployed `.env` file (CI, servers, containers)
- `/yolo` is a Director-initiated override, not a default mode
- Heartbeat and cron-driven sessions must never auto-enable `/yolo`

## Credential Handling

- All credentials are mounted via Docker Compose `env_file` and `:ro` volume mounts
- The agent never decides what credentials get mounted
- `HERMES_YOLO_MODE` must never appear in any `.env` file
- `approvals.mode` must never be set to `"off"` in `config.yaml`
- `HERMES_GATEWAY_SESSION=true` must be set for gateway deployments to ensure the approval system engages

## Supported Versions

This fork tracks the upstream `main` branch. Security fixes are applied on top of each upstream sync.
