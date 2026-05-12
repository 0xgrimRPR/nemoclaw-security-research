# F-15 — Symlink Traversal Blocked [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-15 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Landlock LSM |

## Description

Symlink-based traversal attacks are blocked. Landlock resolves the real target path before enforcing access policy, preventing bypass via symbolic links pointing to restricted locations.

## Evidence

```python
# From /tmp (writable), create symlinks to restricted paths:
>>> os.symlink('/etc/openshell-tls/client/tls.key', '/tmp/link_key')
>>> open('/tmp/link_key').read()
PermissionError  # Landlock resolves → /etc/openshell-tls/client/tls.key → DENIED

>>> os.symlink('/root/.bashrc', '/tmp/link_root')
>>> open('/tmp/link_root').read()
PermissionError  # Resolves → /root/.bashrc → DENIED

>>> os.symlink('/var/run/secrets', '/tmp/link_secrets')
>>> open('/tmp/link_secrets').read()
PermissionError  # Resolves → /var/run/secrets → DENIED
```

## Security Implication

Positive finding. Landlock follows symlink resolution and enforces policy on the real target, preventing a classic sandbox escape technique.
