# F-25 — Device Authentication Disabled by Default [MEDIUM]

| Property | Value |
|----------|-------|
| ID | F-25 |
| Severity | MEDIUM |
| Phase | 4 (May 2026) |
| Component | OpenClaw Gateway Configuration |

## Description

The NemoClaw onboard wizard configures `dangerouslyDisableDeviceAuth: true` and `allowInsecureAuth: true`, eliminating device-level verification. The entire auth chain reduces to a single static token.

## Evidence

```json
"controlUi": {
  "allowInsecureAuth": true,
  "dangerouslyDisableDeviceAuth": true
}
```

## Attack Scenario

1. Gateway token disclosed via prompt injection (F-23) or file read (F-24)
2. No device verification exists to challenge the compromised token
3. Attacker authenticates as full operator with admin scopes

## Security Implication

Single point of failure. Any token disclosure (F-23, F-24) immediately yields full gateway compromise. The flag name itself acknowledges the risk.