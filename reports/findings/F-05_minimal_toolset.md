# F-05 — Minimal Toolset Limits Post-Exploitation Capability [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-05 |
| Severity | INFORMATIONAL |
| Phase | 1 (March 2026) |
| Component | Sandbox container image |

## Description

Common reconnaissance tools are absent from the sandbox image: `ps`, `nslookup`,
`wget`, and `file` are not installed. However, Python3 is present (required for
the blueprint) and provides partial alternatives.

## Evidence

```bash
bash: ps: command not found
bash: nslookup: command not found

python3 -c 'import socket'
→ works
```

## Security Implication

Defense-by-minimalism is effective but not complete. Python3's `socket`,
`subprocess`, `os`, and `ctypes` modules can substitute for several absent tools
during post-exploitation reconnaissance.
