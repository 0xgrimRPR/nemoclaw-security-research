# F-19 — DNS Proxy Change [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-19 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Network / DNS |

## Description

DNS resolution changed from CoreDNS (`10.43.0.10`) observed in Phase 1 to a direct proxy at `10.200.0.1` in the updated NemoClaw v0.0.38. DNS queries now route through the same gateway that handles HTTP proxy traffic.

## Evidence

```
# Phase 1 (v0.1.0):
$ cat /etc/resolv.conf
nameserver 10.43.0.10  ← CoreDNS (K3s default)

# Phase 3 (v0.0.38):
$ cat /etc/resolv.conf
nameserver 10.200.0.1  ← Direct proxy
```

## Security Implication

Architectural change. Consolidating DNS through the gateway proxy reduces the attack surface by eliminating a separate DNS service. All name resolution is now under the same policy enforcement point.
