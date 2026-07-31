# F-02 — K3s Internal Network Unreachable from Sandbox [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-02 |
| Severity | INFORMATIONAL |
| Phase | 1 (March 2026) |
| Component | Network namespace / K3s routing |

## Description

The K3s cluster network (`10.43.0.0/16`), including CoreDNS (`10.43.0.10`) and
the Kubernetes API server, has no route from the sandbox network namespace.

## Evidence

```bash
ip route
→ only 10.200.0.0/24 visible

curl -sk https://10.43.0.1
→ no response (timeout)
```

## Security Implication

Positive finding. The K3s control plane is architecturally isolated. Lateral
movement to the Kubernetes API server via direct network access is not possible
with the current configuration.
