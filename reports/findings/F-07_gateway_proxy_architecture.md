# F-07 — Gateway Proxy Architecture on Port 3128 [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-07 |
| Severity | INFORMATIONAL |
| Phase | 2 (April 2026) |
| Component | OpenShell Gateway |

## Description

The OpenShell gateway operates as a forward HTTP proxy on port 3128 within the sandbox network namespace. Only CONNECT method is accepted. Destinations are validated against a policy whitelist using exact `hostname:port` matching.

## Evidence

```
$ env | grep -i proxy
http_proxy=http://10.200.0.1:3128
https_proxy=http://10.200.0.1:3128
ALL_PROXY=http://10.200.0.1:3128

# Deny response (structured JSON):
HTTP/1.1 403 Forbidden
{"detail":"CONNECT host:port not permitted by policy","error":"policy_denied"}
```

## Security Implication

Positive finding. Clean enforcement point for network policy. CONNECT-only restriction limits attack surface.
