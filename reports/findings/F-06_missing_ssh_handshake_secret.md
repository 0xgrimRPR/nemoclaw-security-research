# F-06 — OpenShell 0.0.7 Missing K8s Secret During Onboard [BUG]

| Property | Value |
|----------|-------|
| ID | F-06 |
| Severity | BUG |
| Phase | 2 (April 2026) |
| Component | OpenShell 0.0.7 / Helm chart |
| Status | Reproducible |

## Description

The OpenShell gateway provisioning process (`gateway start`) and NemoClaw onboarding (`nemoclaw onboard`) fail to create the Kubernetes secret `openshell-ssh-handshake` required by the OpenShell Helm chart. The pod `openshell-0` enters `CreateContainerConfigError` state and loops indefinitely.

This bug is **reproducible**: destroying and recreating the gateway triggers the same failure every time.

## Evidence

```
$ docker logs openshell-cluster-openshell --tail 5
E0416 kuberuntime_manager.go:1664] "Unhandled Error"
  err="container openshell start failed in pod openshell-0_openshell:
  CreateContainerConfigError: secret \"openshell-ssh-handshake\" not found"

E0416 pod_workers.go:1324] "Error syncing pod, skipping"
  err="failed to \"StartContainer\" for \"openshell\" with
  CreateContainerConfigError: \"secret \\\"openshell-ssh-handshake\\\" not found\""

$ kubectl get pods -n openshell
NAME          READY   STATUS                       RESTARTS   AGE
openshell-0   0/1     CreateContainerConfigError   0          8m
```

## Root Cause

The StatefulSet spec references an environment variable sourced from a secret:

```yaml
- name: OPENSHELL_SSH_HANDSHAKE_SECRET
  valueFrom:
    secretKeyRef:
      key: secret
      name: openshell-ssh-handshake
```

The Helm chart expects this secret to exist but neither `openshell gateway start` nor `nemoclaw onboard` creates it.

## Workaround

```bash
docker exec openshell-cluster-openshell kubectl create secret generic \
  openshell-ssh-handshake \
  --from-literal=secret="$(openssl rand -hex 32)" \
  -n openshell

docker exec openshell-cluster-openshell kubectl delete pod openshell-0 \
  -n openshell
```

## Impact

Sandbox cannot start without manual intervention. Blocking defect for new users.
