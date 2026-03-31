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

### 2. YOLO Mode Removed (`tools/approval.py`)

**Upstream behavior:** Setting `HERMES_YOLO_MODE` environment variable disables ALL approval and security checks globally.

**Why this is unsafe:** A single environment variable typo or misconfiguration collapses the entire security model. The `approvals.mode: "off"` config path has also been removed from the combined guard for the same reason.

**Fix:** Removed the `os.getenv("HERMES_YOLO_MODE")` checks in both `check_dangerous_command()` and `check_all_command_guards()`. Removed the `approval_mode == "off"` bypass in `check_all_command_guards()`.

## Credential Handling

- All credentials are mounted via Docker Compose `env_file` and `:ro` volume mounts
- The agent never decides what credentials get mounted
- `HERMES_YOLO_MODE` must never appear in any `.env` file
- `approvals.mode` must never be set to `"off"` in `config.yaml`
- `HERMES_GATEWAY_SESSION=true` must be set for gateway deployments to ensure the approval system engages

## Supported Versions

This fork tracks the upstream `main` branch. Security fixes are applied on top of each upstream sync.
