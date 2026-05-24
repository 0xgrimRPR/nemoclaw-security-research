# F-24 — Operator Token and Private Key Accessible Within Sandbox [MEDIUM]

| Property | Value |
|----------|-------|
| ID | F-24 |
| Severity | MEDIUM |
| Phase | 4 (May 2026) |
| Component | OpenClaw Identity Storage |

## Description

The sandbox user has read access to operator tokens, gateway auth tokens, and Ed25519 private keys stored in plaintext. Any code execution within the sandbox can harvest credentials without privilege escalation.

## Evidence

```
~/.openclaw/identity/device-auth.json  → Operator token (admin scopes)
~/.openclaw/identity/device.json       → Ed25519 private key (PEM)
~/.openclaw/openclaw.json              → Gateway auth token (plaintext)
~/.openclaw/devices/paired.json        → Operator token + approved scopes
```

All files readable by sandbox user. Extends F-20 (agent state writable) with credential-specific impact.

## Attack Scenario

1. Prompt injection (F-23) directs agent to read identity files
2. Malicious npm/pip package reads files via postinstall hook
3. Any code execution within sandbox harvests credentials
4. Credentials exfiltrated via allowed egress (github.com, npmjs.org)

## Security Implication

Plaintext credential storage within agent reach. Prompt injection provides a no-code-execution path to credential theft.