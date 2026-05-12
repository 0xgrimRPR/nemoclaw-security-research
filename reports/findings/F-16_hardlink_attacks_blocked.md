# F-16 — Hardlink Attacks Blocked [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-16 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Kernel / Filesystem |

## Description

Hardlink-based attacks are blocked by two independent mechanisms: cross-device link restrictions (overlay filesystem boundaries) and kernel-level `protected_hardlinks` sysctl.

## Evidence

```python
>>> os.link('/etc/openshell-tls/client/tls.key', '/tmp/hard_key')
OSError: [Errno 18] Invalid cross-device link  # Different filesystem

>>> os.link('/etc/passwd', '/tmp/hard_passwd')
PermissionError: [Errno 1] Operation not permitted  # Kernel protection

>>> os.link('/etc/openshell-tls/openshell-ca.pem', '/tmp/hard_ca')
PermissionError: [Errno 1] Operation not permitted  # Kernel protection
```

## Security Implication

Positive finding. Defense in depth — two independent mechanisms prevent hardlink attacks. Even if one fails, the other provides protection.
