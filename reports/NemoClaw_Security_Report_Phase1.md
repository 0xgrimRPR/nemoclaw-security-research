# NemoClaw Security Assessment — Phase 1 Report

**Black-box Reconnaissance & Architecture Mapping**

| Field | Value |
|-------|-------|
| Date | March 17, 2026 |
| Analyst | 0xgrimRPR |
| Role | Junior Cybersecurity Engineer / Red Team Track |
| Target | NVIDIA NemoClaw 0.0.x (Early Preview — Alpha) |
| Environment | VMware VM / Ubuntu 22.04 LTS / No GPU |
| Scope | Black-box reconnaissance & architecture mapping |
| Classification | Research Use Only |

---

## 1. Executive Summary

This report documents a black-box security reconnaissance session conducted against NVIDIA NemoClaw, an open-source AI agent platform announced at GTC 2026 on March 16, 2026. The assessment was performed within 24 hours of the platform's public release — before any public CVEs or security advisories existed.

NemoClaw is a stack built on OpenClaw (the fastest-growing open-source AI agent project in GitHub history) that adds privacy and security guardrails via NVIDIA OpenShell. The platform runs AI agents inside sandboxed environments using multiple defense-in-depth mechanisms.

**Overall assessment:** The security posture is well-designed for alpha-stage software. No critical vulnerabilities or credential leaks were identified. All seven defense layers functioned as intended. One notable observation involves a Kubernetes service account token that is currently mitigated by Landlock but warrants ongoing monitoring.

---

## 2. Test Environment

### 2.1 Infrastructure

| Component | Details |
|-----------|---------|
| Host OS | Ubuntu 22.04 LTS (VMware Virtual Platform) |
| Virtualization | VMware (no GPU passthrough) |
| RAM | 7.7 GiB + 8 GiB swap |
| CPU | 4 cores (x86_64) |
| Docker | Running, socket accessible to sandbox user group |
| Node.js | v22.20.1 |
| OpenShell | v0.0.7 (installed via PyPI / uv) |
| NemoClaw | 0.0.x early preview (cloned from GitHub NVIDIA/NemoClaw) |
| Inference | NVIDIA Cloud API — Nemotron 3 Super 120B |
| Sandbox name | cortana |

### 2.2 Installation Path

1. Ubuntu 22.04 base VM with Docker installed and enabled
2. Node.js v22 via NodeSource repository (v20 was insufficient — NemoClaw requires >=22.16.0)
3. npm globals directory configured in `~/.npm-global` to avoid permission issues
4. NVIDIA OpenShell v0.0.7 installed via: `uv tool install -U openshell`
5. NemoClaw cloned from GitHub and installed from source via `./install.sh`
6. Sandbox 'cortana' provisioned via `nemoclaw onboard` interactive wizard
7. Policy presets applied: `pypi + npm` (suggested defaults accepted)

---

## 3. Discovered Architecture

Reconnaissance revealed a multi-layer containerized architecture that was not fully documented at time of assessment:

```
[Host VM: Ubuntu 22.04]
└── Docker container (OpenShell Gateway)
    └── K3s cluster (Kubernetes lightweight)
        ├── CoreDNS (10.43.0.10)
        ├── Inference proxy (inference.local)
        └── Sandbox pod 'cortana'  ← assessment entry point
            IP: 10.200.0.2/24
            Gateway: 10.200.0.1 (OpenShell proxy)
            → NVIDIA Cloud API (internet egress)
```

The sandbox is a Kubernetes pod running inside a K3s cluster, which itself runs inside a Docker container. This Kubernetes-in-Docker pattern was confirmed by `/etc/resolv.conf` pointing to CoreDNS at `10.43.0.10` with the `svc.cluster.local` search domain. The K3s control plane network (`10.43.0.0/16`) has no route from the sandbox network namespace, providing strong isolation.

---

## 4. Security Layers Analysis

### 4.1 Defense-in-Depth Summary

| Layer | Mechanism | Status | Effectiveness |
|-------|-----------|--------|---------------|
| Filesystem | Landlock LSM (best_effort) | ACTIVE | High |
| Syscalls | seccomp BPF (mode 2, 1 filter) | ACTIVE | High |
| Privileges | Linux Capabilities (CapEff=0) | ACTIVE | High |
| Network egress | netns + OpenShell proxy | ACTIVE | High |
| K3s isolation | Routing table restriction | ACTIVE | High |
| Toolset | Minimal container image | ACTIVE | Medium |
| Credentials | No hardcoded secrets | VERIFIED | High |

### 4.2 Filesystem — Landlock LSM

The sandbox enforces filesystem restrictions via Linux Landlock Security Module configured in `best_effort` compatibility mode. Write access is limited to `/sandbox` and `/tmp`. All system paths (`/etc`, `/usr`, `/lib`, `/proc`) are mounted read-only. The Kubernetes service account token is present but unreadable.

```bash
touch /etc/test.txt       → Permission denied
wc -c .../token           → Permission denied
```

### 4.3 Syscall Filtering — seccomp BPF

The sandbox user process runs with seccomp mode 2 (BPF filter active). PID 1 shows `Seccomp: 0` because the container init starts without the filter and applies it to spawned processes — standard Docker behavior. One BPF filter program is loaded.

```bash
cat /proc/self/status | grep Seccomp
Seccomp: 2
Seccomp_filters: 1
```

### 4.4 Linux Capabilities

The sandbox user process has zero effective and permitted capabilities. No privileged operations are possible even if a vulnerability is exploited. PID 1 (root) holds a reduced capability set that excludes `CAP_SYS_ADMIN`, `CAP_SYS_MODULE`, and `CAP_NET_RAW`.

```
CapPrm (sandbox user): 0000000000000000
CapEff (sandbox user): 0000000000000000
CapEff (PID 1 root):   00000004a82c35fb
```

### 4.5 Network Isolation

The sandbox runs in an isolated network namespace with a single veth interface (`10.200.0.2/24`). The gateway at `10.200.0.1` intercepts all traffic. The K3s cluster network has no route from the sandbox, blocking all access to the Kubernetes control plane.

```bash
ip route: default via 10.200.0.1 dev veth-s-719f5913
curl https://google.com  → CONNECT tunnel failed, response 403
```

### 4.6 Network Policy Whitelist

| Policy Name | Allowed Endpoints |
|-------------|-------------------|
| nvidia | integrate.api.nvidia.com:443, inference-api.nvidia.com:443 |
| clawhub | clawhub.com:443 (GET + POST only) |
| openclaw_api | openclaw.ai:443 (GET + POST only) |
| openclaw_docs | docs.openclaw.ai:443 (GET only) |
| github | github.com:443, api.github.com:443 (full) |
| claude_code | api.anthropic.com:443, statsig.anthropic.com:443 |
| npm_registry | registry.npmjs.org:443 |
| telegram | api.telegram.org:443 (bot paths only) |

Each policy entry also specifies a binaries allowlist — only specific executables (`openclaw`, `git`, `gh`, `npm`) can reach their respective endpoints.

---

## 5. Security Findings

### F-01 — K8s Service Account Token Mounted but Inaccessible [LOW]

**Description:** The Kubernetes service account token is automatically mounted by K3s at `/var/run/secrets/kubernetes.io/serviceaccount/token` inside the sandbox pod. The file exists and the symlink chain is intact, but Landlock prevents the sandbox user from reading it.

**Evidence:**
```bash
ls -la /var/run/secrets/.../serviceaccount/
token -> ..data/token  [EXISTS]

wc -c token
Permission denied
```

**Security Implication:** Mitigated by Landlock. If Landlock were bypassed or misconfigured in a future version, the token could grant authenticated access to the K3s API server. Recommend monitoring this path across version updates.

---

### F-02 — K3s Internal Network Unreachable from Sandbox [INFORMATIONAL]

**Description:** The K3s cluster network (`10.43.0.0/16`) including CoreDNS (`10.43.0.10`) and the Kubernetes API server have no route from the sandbox network namespace.

**Evidence:**
```bash
ip route
→ only 10.200.0.0/24 visible

curl -sk https://10.43.0.1
→ no response (timeout)
```

**Security Implication:** Positive finding. The K3s control plane is architecturally isolated. Lateral movement to the Kubernetes API server via direct network access is not possible with the current configuration.

---

### F-03 — PID 1 Runs as Root with Non-Zero Capabilities [INFORMATIONAL]

**Description:** The init process (`openshell-sandb`, PID 1) runs as UID 0 with `CapEff: 0x00000004a82c35fb`. This is expected behavior for a container init managing the sandbox lifecycle.

**Evidence:**
```bash
cat /proc/1/status
Uid: 0 0 0 0
CapEff: 00000004a82c35fb
```

**Security Implication:** Standard containerization pattern. The sandbox user inherits zero capabilities. The PID 1 capability set should be audited in future releases to verify minimum required capabilities.

---

### F-04 — No Hardcoded Credentials in Distributed Code [INFORMATIONAL]

**Description:** Static analysis of `/opt/nemoclaw/dist/` found no hardcoded API keys, secrets, passwords, or tokens. NVIDIA API key management is delegated to the OpenShell gateway at runtime.

**Evidence:**
```bash
grep -r 'api_key|secret|password' /opt/nemoclaw/dist/
→ Only variable names in JS logic, no credential values
```

**Security Implication:** Positive finding. Credentials are stored in `~/.nemoclaw/credentials.json` (mode 600) on the host, not embedded in the agent image.

---

### F-05 — Minimal Toolset Limits Post-Exploitation Capability [INFORMATIONAL]

**Description:** Common reconnaissance tools are absent from the sandbox image: `ps`, `nslookup`, `wget`, `file` are not installed. However, Python3 is present (required for the blueprint) and provides partial alternatives.

**Evidence:**
```bash
bash: ps: command not found
bash: nslookup: command not found

python3 -c 'import socket'
→ works
```

**Security Implication:** Defense-by-minimalism is effective but not complete. Python3's `socket`, `subprocess`, `os`, and `ctypes` modules can substitute for several absent tools during post-exploitation reconnaissance.

---

## 6. Residual Attack Surface

The following areas represent research directions for deeper assessment sessions:

### 6.1 OpenShell Gateway (10.200.0.1)
All sandbox traffic transits the gateway. A vulnerability in the gateway process could allow traffic manipulation or policy bypass without touching the sandbox. Gateway port surface and exposed admin endpoints have not been assessed.

### 6.2 Binary Allowlist Enforcement Mechanism
Network policies restrict endpoints by binary path (e.g., only `/usr/local/bin/openclaw` can reach `openclaw.ai`). If enforcement relies on process name rather than inode or cryptographic verification, binary substitution attacks may be viable.

### 6.3 Python3 as Post-Exploitation Substitute
```python
python3 -c "import socket; print(socket.gethostbyname('inference.local'))"
```
Python3 is installed and provides `socket`, `subprocess`, `ctypes`, and `os` — partial replacements for absent system tools.

### 6.4 /opt/nemoclaw World-Readable
`/opt/nemoclaw` is owned by root but world-readable. The `dist/` directory contains the full TypeScript-compiled plugin source. While no credentials were found, future versions should review what is bundled into the image.

### 6.5 Prompt Injection via Agent Input
As a platform running an autonomous AI agent, NemoClaw inherits the full prompt injection risk surface of OpenClaw. Attacker-controlled input to the agent could manipulate behavior within the bounds of the network policy. This has not been tested and is planned for Phase 4.

---

## 7. Available Inference Models

Four models discovered in the NemoClaw source (`onboard.js`):

| Model ID | Label |
|----------|-------|
| nvidia/nemotron-3-super-120b-a12b | Nemotron 3 Super 120B (default) |
| nvidia/llama-3.1-nemotron-ultra-253b-v1 | Nemotron Ultra 253B |
| nvidia/llama-3.3-nemotron-super-49b-v1.5 | Nemotron Super 49B v1.5 |
| nvidia/nemotron-3-nano-30b-a3b | Nemotron 3 Nano 30B (low-RAM) |

---

## 8. Conclusions

NVIDIA NemoClaw demonstrates a thoughtful defense-in-depth approach for AI agent sandboxing. The combination of Landlock, seccomp BPF, zero-capability user processes, and network namespace isolation creates a meaningful security boundary around autonomous agent execution.

For alpha software assessed within 24 hours of public release, the security architecture is notably mature. The primary risks are not in the sandbox controls themselves but in the gateway layer attack surface and the inherent prompt injection risks of autonomous AI agents — both areas planned for the next session.

Early reconnaissance of emerging AI infrastructure is high-value work. Security patterns established in alpha are typically carried forward to production. Documenting them now informs both defensive hardening recommendations and future red team exercises.

---

*Report generated: March 17, 2026 | NemoClaw 0.0.x (alpha) | For research purposes only*
