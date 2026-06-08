# NemoClaw Phase 3 — Filesystem / Landlock Boundary Testing

| Field | Value |
|-------|-------|
| Date | May 9, 2026 |
| Analyst | 0xgrimRPR |
| Target | NemoClaw v0.0.38 + OpenShell 0.0.36 |

## Summary

Phase 3 confirmed Landlock LSM provides comprehensive filesystem boundary enforcement. Root directory not listable, symlink/hardlink attacks blocked, mTLS key protected. Only /tmp and /sandbox writable. Ten findings (F-13 to F-22).

## Filesystem Access

| Path | Access |
|------|--------|
| `/` | DENIED |
| `/etc` | READ (95 entries) |
| `/tmp` | READ/WRITE (1,280 files) |
| `/sandbox` | READ/WRITE |
| `/usr` | READ |
| `/home, /var, /opt, /run, /root` | DENIED |

## Bypass Tests

- Symlink traversal: BLOCKED (Landlock follows real target)
- Hardlink attack: BLOCKED (cross-device + kernel protection)
- /proc/1/environ: DENIED (cmdline/status readable)
- Safety script write: BLOCKED (read-only)

## Security Controls

- CapEff: 0 | NoNewPrivs: 1 | Seccomp filters: 3
- Node.js safety scripts injected via NODE_OPTIONS (read-only)
- Agent runtime state in /sandbox/.openclaw/ is writable (Phase 4 target)

## Findings

| ID | Finding | Severity |
|----|---------|----------|
| F-13 | Root dir not listable | INFORMATIONAL |
| F-14 | Environ no secrets | INFORMATIONAL |
| F-15 | Symlink blocked | INFORMATIONAL |
| F-16 | Hardlink blocked | INFORMATIONAL |
| F-17 | PID 1 info leak | LOW |
| F-18 | Safety scripts read-only | INFORMATIONAL |
| F-19 | DNS proxy change | INFORMATIONAL |
| F-20 | Agent state writable | LOW |
| F-21 | Seccomp 3 filters | INFORMATIONAL |
| F-22 | NoNewPrivs enforced | INFORMATIONAL |

## Cumulative Findings (F-01 to F-22)

| ID | Finding | Severity | Phase |
|----|---------|----------|-------|
| F-01 | K8s Token Mounted but Inaccessible | LOW | 1 |
| F-02 | K3s Network Unreachable | INFORMATIONAL | 1 |
| F-03 | PID 1 Root with Capabilities | INFORMATIONAL | 1 |
| F-04 | No Hardcoded Credentials | INFORMATIONAL | 1 |
| F-05 | Minimal Toolset | INFORMATIONAL | 1 |
| F-06 | Missing K8s Secret | BUG | 2 |
| F-07 | Gateway Proxy on 3128 | INFORMATIONAL | 2 |
| F-08 | Proxy Bypass Resistant | INFORMATIONAL | 2 |
| F-09 | Data Channel via githubusercontent | LOW | 2 |
| F-10 | TLS MITM via OpenShell CA | MEDIUM | 2 |
| F-11 | mTLS Key Protected | INFORMATIONAL | 2 |
| F-12 | Non-CONNECT Blocked | INFORMATIONAL | 2 |
| F-13 | Root Dir Not Listable | INFORMATIONAL | 3 |
| F-14 | Environ No Secrets | INFORMATIONAL | 3 |
| F-15 | Symlink Blocked | INFORMATIONAL | 3 |
| F-16 | Hardlink Blocked | INFORMATIONAL | 3 |
| F-17 | PID 1 Info Leak | LOW | 3 |
| F-18 | Safety Scripts Read-Only | INFORMATIONAL | 3 |
| F-19 | DNS Proxy Change | INFORMATIONAL | 3 |
| F-20 | Agent State Writable | LOW | 3 |
| F-21 | Seccomp 3 Filters | INFORMATIONAL | 3 |
| F-22 | NoNewPrivs Enforced | INFORMATIONAL | 3 |

*Report: May 9, 2026 | For research purposes only*
