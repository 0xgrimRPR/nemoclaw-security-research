# F-03 — PID 1 Runs as Root with Non-Zero Capabilities [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-03 |
| Severity | INFORMATIONAL |
| Phase | 1 (March 2026) |
| Component | Container init / Linux Capabilities |

## Description

The init process (`openshell-sandb`, PID 1) runs as UID 0 with
`CapEff: 0x00000004a82c35fb`. This is expected behavior for a container init
managing the sandbox lifecycle.

## Evidence

```bash
cat /proc/1/status
Uid: 0 0 0 0
CapEff: 00000004a82c35fb
```

## Security Implication

Standard containerization pattern. The sandbox user inherits zero capabilities.
The PID 1 capability set should be audited in future releases to verify only the
minimum required capabilities are retained.
