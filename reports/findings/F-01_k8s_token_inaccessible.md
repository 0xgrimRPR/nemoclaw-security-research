# F-01 — K8s Service Account Token Mounted but Inaccessible [LOW]

| Property | Value |
|----------|-------|
| ID | F-01 |
| Severity | LOW |
| Phase | 1 (March 2026) |
| Component | Landlock LSM / K3s service account |

## Description

The Kubernetes service account token is automatically mounted by K3s at
`/var/run/secrets/kubernetes.io/serviceaccount/token` inside the sandbox pod.
The file exists and the symlink chain is intact, but Landlock prevents the
sandbox user from reading it.

## Evidence

```bash
ls -la /var/run/secrets/.../serviceaccount/
token -> ..data/token  [EXISTS]

wc -c token
Permission denied
```

## Security Implication

Mitigated by Landlock. If Landlock were bypassed or misconfigured in a future
version, the token could grant authenticated access to the K3s API server.
Recommend monitoring this path across version updates.
