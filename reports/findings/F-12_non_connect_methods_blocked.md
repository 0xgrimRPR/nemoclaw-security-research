# F-12 — Non-CONNECT HTTP Methods Blocked at Proxy [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-12 |
| Severity | INFORMATIONAL |
| Phase | 2 (April 2026) |
| Component | OpenShell Gateway Proxy |

## Description

The proxy rejects all non-CONNECT HTTP methods with `403 Forbidden`.

## Evidence

```
GET http://github.com/ HTTP/1.1    → 403 Forbidden
POST http://github.com/ HTTP/1.1   → 403 Forbidden
PUT http://github.com/ HTTP/1.1    → 403 Forbidden
DELETE http://github.com/ HTTP/1.1 → 403 Forbidden
```

## Security Implication

Positive finding. Ensures all egress uses CONNECT tunnels subject to TLS MITM inspection.
