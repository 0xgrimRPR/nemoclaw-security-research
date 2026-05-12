# F-13 — Root Dir Not Listable [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-13 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Landlock LSM |

## Description

Landlock blocks `os.listdir('/')`, preventing enumeration of the root filesystem. The sandbox process cannot discover top-level directory structure through standard directory listing calls.

## Evidence

```python
>>> os.listdir('/')
PermissionError: [Errno 13] Permission denied: '/'
```

Accessible directories were discovered individually by probing known paths:

| Path | Status |
|------|--------|
| `/etc` | READ (95 entries) |
| `/tmp` | READ/WRITE |
| `/sandbox` | READ/WRITE |
| `/usr` | READ |
| `/home, /var, /opt, /run, /root` | DENIED |

## Security Implication

Positive finding. Limits reconnaissance capability within the sandbox by preventing filesystem discovery from the root level.
