# F-22 — NoNewPrivs Enforced [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-22 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Kernel / Privilege Controls |

## Description

The `NoNewPrivs` bit is set to `1`, preventing the sandbox process (and any children) from gaining additional privileges via `execve()`. This blocks setuid/setgid binaries and other privilege escalation vectors at the kernel level.

## Evidence

```
$ cat /proc/self/status | grep NoNewPrivs
NoNewPrivs:	1

$ cat /proc/self/status | grep -i cap
CapInh:  0000000000000000
CapPrm:  0000000000000000
CapEff:  0000000000000000
CapBnd:  00000004a82c35fb
CapAmb:  0000000000000000
```

Zero effective, permitted, and inheritable capabilities combined with `NoNewPrivs` creates a strong privilege boundary.

## Security Implication

Positive finding. Privilege escalation is blocked at the kernel level. Even if a setuid binary existed in the sandbox, it could not gain elevated privileges.
