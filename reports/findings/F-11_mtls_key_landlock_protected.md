# F-11 — mTLS Client Key Protected by Landlock [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-11 |
| Severity | INFORMATIONAL |
| Phase | 2 (April 2026) |
| Component | Landlock / TLS |

## Description

The sandbox-to-gateway mTLS client key at `/etc/openshell-tls/client/tls.key` (241 bytes, ECDSA) is protected by Landlock.

## Evidence

```
$ cat /etc/openshell-tls/client/tls.key
Permission denied
```

## Security Implication

Positive finding. Key correctly protected. Extraction requires Landlock bypass (Phase 3 scope).
