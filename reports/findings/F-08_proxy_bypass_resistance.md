# F-08 — Proxy Resistant to Common Bypass Techniques [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-08 |
| Severity | INFORMATIONAL |
| Phase | 2 (April 2026) |
| Component | OpenShell Gateway Proxy |

## Description

Ten bypass techniques were tested against the proxy policy engine. All were blocked or mitigated.

## Techniques Tested

| # | Technique | Result |
|---|-----------|--------|
| 1 | Host header spoofing | BLOCKED |
| 2 | IP-based CONNECT | BLOCKED |
| 3 | Non-standard ports | BLOCKED |
| 4 | Subdomain wildcarding | PARTIAL (objects.githubusercontent.com allowed) |
| 5 | SNI mismatch post-CONNECT | MITIGATED (by destination server) |
| 6 | Post-CONNECT internal reach | BLOCKED |
| 7 | Double CONNECT (nested) | BLOCKED |
| 8 | Non-CONNECT methods | BLOCKED |
| 9 | HTTP redirect following | BLOCKED |
| 10 | mTLS key extraction | BLOCKED (Landlock) |

## Security Implication

Positive finding. Robust policy enforcement confirmed.
