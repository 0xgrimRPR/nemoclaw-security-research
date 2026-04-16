# F-09 — Data Channel via objects.githubusercontent.com [LOW]

| Property | Value |
|----------|-------|
| ID | F-09 |
| Severity | LOW |
| Phase | 2 (April 2026) |
| Component | Network Policy / GitHub preset |

## Description

The GitHub policy permits CONNECT to `objects.githubusercontent.com:443`, which serves binary assets via Varnish CDN. This creates a potential bidirectional data channel for payload delivery and exfiltration.

## Evidence

```
CONNECT objects.githubusercontent.com:443 → 200 Connection Established
GET / → HTTP/1.1 404 Not Found
Server: Varnish
X-Served-By: cache-iah-kiah1900045-IAH
```

## Attack Scenario

1. Attacker creates a GitHub repo with release assets containing payloads
2. From sandbox: download payloads via `objects.githubusercontent.com`
3. Exfiltrate data by pushing to repo via `api.github.com`

## Mitigation

Inherent to allowing GitHub access. Mitigated by TLS MITM visibility. Consider monitoring unusual download patterns.
