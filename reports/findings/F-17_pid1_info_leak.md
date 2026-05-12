# F-17 — PID 1 Info Leak [LOW]

| Property | Value |
|----------|-------|
| ID | F-17 |
| Severity | LOW |
| Phase | 3 (May 2026) |
| Component | procfs / Process Isolation |

## Description

PID 1 (`/opt/openshell/bin/openshell-sandbox`) exposes its command line, cgroup membership, and process status to the sandbox process. Sensitive data (environ, memory maps) is protected.

## Evidence

```
$ cat /proc/1/cmdline
/opt/openshell/bin/openshell-sandbox

$ cat /proc/1/status
Name: openshell-san
State: S (sleeping)
Threads: [readable]

$ cat /proc/1/cgroup
0::/

$ cat /proc/1/environ
Permission denied  ← PROTECTED

$ cat /proc/1/maps
Permission denied  ← PROTECTED
```

## Security Implication

Low-severity information disclosure. Reveals the supervisor binary path and process state, which aids reconnaissance. Sensitive data remains protected.
