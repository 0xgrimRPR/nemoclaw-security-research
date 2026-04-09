# Black-Box Reconnaissance Methodology

This document describes the approach used for Phase 1 of the NemoClaw security assessment. It is intended to be reproducible — anyone with the same setup should be able to follow these steps and verify the findings.

---

## Guiding Principles

**No prior documentation assumed.** The assessment started with only what was publicly available at release: the GitHub repository and the install instructions. Internal architecture was inferred entirely from observable behavior.

**Evidence before conclusions.** Every claim in the reports is backed by a command and its output. If there is no output, there is no finding.

**Offense informs defense.** The goal is not to break NemoClaw — it is to understand what an attacker would discover in the same position, so that the architecture can be hardened before it reaches production use.

---

## Reconnaissance Approach

### Step 1 — Environment fingerprinting

Before touching the sandbox itself, establish what is known about the host environment:

```bash
uname -a           # kernel version and architecture
cat /etc/os-release
id                 # current user and groups
docker info        # docker socket accessibility
```

### Step 2 — Sandbox entry and identity

Once inside the sandbox (via `nemoclaw run`), establish basic identity:

```bash
id                         # UID, GID, groups
cat /proc/self/status      # capabilities, seccomp mode
cat /proc/1/status         # PID 1 identity and capabilities
```

### Step 3 — Network topology mapping

Understand where the sandbox sits in the network:

```bash
ip addr
ip route
cat /etc/resolv.conf       # DNS reveals K3s/CoreDNS presence
cat /etc/hosts
```

Test egress behavior:

```bash
curl -v https://google.com           # expect 403 from proxy
curl -v https://api.anthropic.com    # test whitelisted endpoint
```

### Step 4 — Filesystem boundary testing

Map what is accessible vs. restricted:

```bash
# Write access test
touch /etc/test.txt
touch /tmp/test.txt
touch /sandbox/test.txt

# Credential access test
ls -la /var/run/secrets/kubernetes.io/serviceaccount/
wc -c /var/run/secrets/kubernetes.io/serviceaccount/token

# Static analysis
find /opt -readable -type f 2>/dev/null | head -50
grep -r 'api_key\|secret\|password' /opt/nemoclaw/dist/
```

### Step 5 — Process and capability analysis

Understand the privilege model:

```bash
cat /proc/self/status | grep -E 'Cap|Seccomp'
cat /proc/1/status | grep -E 'Cap|Uid'
ls /proc/ | head -20      # visible PIDs
```

### Step 6 — Toolset inventory

Understand what is available for post-exploitation:

```bash
which python3 python node npm git curl wget ps nslookup file
python3 -c 'import socket, subprocess, os, ctypes; print("all available")'
```

---

## What This Methodology Does Not Cover

- **Source code review** — the gateway source is not publicly available; this is purely behavioral analysis
- **Fuzzing** — not performed in Phase 1
- **Exploit development** — Phase 1 is reconnaissance only; no exploitation was attempted
- **Network traffic capture** — would require host-level access not available in the sandbox

These are planned for later phases where warranted by findings.

---

*Last updated: March 2026*
