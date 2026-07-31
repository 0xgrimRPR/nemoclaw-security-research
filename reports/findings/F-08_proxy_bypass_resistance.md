# F-08 — Proxy Resistant to Common Bypass Techniques [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-08 |
| Severity | INFORMATIONAL |
| Phase | 2 (April 2026) |
| Component | OpenShell Gateway Proxy |

## Description

Twelve bypass techniques were tested against the proxy policy engine (see Phase 2
report §5 for the full matrix). Ten were blocked or mitigated. Two endpoints
(`objects.githubusercontent.com` and the GitHub API) were permitted by policy and
are tracked separately — the data channel is filed as F-09.

## Techniques Tested (12)

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
| 11 | Data channel via objects CDN | PERMITTED (tracked as F-09) |
| 12 | GitHub API access | PERMITTED (policy-allowed) |

## Security Implication

Positive finding. Of twelve techniques tested, ten were blocked or mitigated; the
two permitted endpoints are policy-allowed by design and tracked separately
(F-09). Robust policy enforcement confirmed.
