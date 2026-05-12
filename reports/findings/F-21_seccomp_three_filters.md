# F-21 — Seccomp 3 Filters [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-21 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Seccomp BPF |

## Description

The sandbox now runs with 3 seccomp BPF filters, up from 2 observed in Phase 1. This indicates progressive syscall hardening between NemoClaw versions.

## Evidence

```
$ cat /proc/self/status | grep -i seccomp
Seccomp:         2
Seccomp_filters: 3
```

Phase 1 (v0.1.0) reported `Seccomp_filters: 2`.

## Security Implication

Positive finding. Additional seccomp filter layer suggests NVIDIA is actively tightening syscall restrictions across releases.
