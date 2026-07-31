# F-04 — No Hardcoded Credentials in Distributed Code [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-04 |
| Severity | INFORMATIONAL (positive) |
| Phase | 1 (March 2026) |
| Component | NemoClaw dist / credential handling |

## Description

Static analysis of `/opt/nemoclaw/dist/` found no hardcoded API keys, secrets,
passwords, or tokens. NVIDIA API key management is delegated to the OpenShell
gateway at runtime.

## Evidence

```bash
grep -r 'api_key|secret|password' /opt/nemoclaw/dist/
→ Only variable names in JS logic, no credential values
```

## Security Implication

Positive finding. Credentials are stored in `~/.nemoclaw/credentials.json`
(mode 600) on the host, not embedded in the agent image.
