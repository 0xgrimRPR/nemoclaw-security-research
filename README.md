# NemoClaw Security Research

**Black-box security assessment of NVIDIA NemoClaw v0.1.0 — AI agent sandbox architecture analysis**

> Conducted within 24 hours of the platform's public release at GTC 2026 (March 16, 2026).  
> This research is ongoing. Phases 1–2 are complete. Phases 3–4 are in progress.

---

## Why This Matters

NemoClaw is NVIDIA's answer to the emerging challenge of running autonomous AI agents safely. Built on OpenClaw — one of the fastest-growing open-source AI agent projects in GitHub history — it layers privacy and security controls via NVIDIA OpenShell.

As AI agent platforms become critical infrastructure, understanding their security architecture from the outside is increasingly valuable. This research documents what a black-box assessor can discover, infer, and test against a freshly released AI sandbox — without documentation, without source-level access to the security layer, and without vendor guidance.

This type of work aligns directly with what industry initiatives like [Anthropic's Project Glasswing](https://www.anthropic.com) are validating: that hands-on AI infrastructure security research, not just theory, is what moves the needle on AI safety.

---

## Research Status

| Phase | Focus | Status |
|-------|-------|--------|
| Phase 1 | Architecture mapping & defense layer enumeration | ✅ Complete |
| Phase 2 | Gateway analysis & network policy bypass testing | ✅ Complete |
| Phase 3 | Filesystem / Landlock boundary testing | 🔜 Planned |
| Phase 4 | Prompt injection & agent behavior manipulation | 🔜 Planned |

---

## Key Findings

### Phase 1 — Reconnaissance (March 2026)

Five findings documented across the initial reconnaissance session:

| ID | Title | Severity |
|----|-------|----------|
| F-01 | K8s service account token mounted but inaccessible | LOW |
| F-02 | K3s internal network unreachable from sandbox | INFORMATIONAL |
| F-03 | PID 1 runs as root with non-zero capabilities | INFORMATIONAL |
| F-04 | No hardcoded credentials in distributed code | INFORMATIONAL (positive) |
| F-05 | Minimal toolset limits post-exploitation capability | INFORMATIONAL |

Full details in [`/reports/NemoClaw_Security_Report_Phase1.md`](/reports/NemoClaw_Security_Report_Phase1.md).

### Phase 2 — Gateway & Network Policy (April 2026)

Seven findings documented across gateway analysis and bypass testing:

| ID | Title | Severity |
|----|-------|----------|
| F-06 | OpenShell 0.0.7 missing K8s secret during onboard | BUG |
| F-07 | Gateway proxy architecture on port 3128 | INFORMATIONAL |
| F-08 | Proxy resistant to common bypass techniques | INFORMATIONAL (positive) |
| F-09 | Data channel via objects.githubusercontent.com | LOW |
| F-10 | TLS MITM via OpenShell Sandbox CA | MEDIUM |
| F-11 | mTLS client key protected by Landlock | INFORMATIONAL (positive) |
| F-12 | Non-CONNECT HTTP methods blocked at proxy | INFORMATIONAL (positive) |

Full details in [`/reports/NemoClaw_Security_Report_Phase2.md`](/reports/NemoClaw_Security_Report_Phase2.md).

---

## Architecture Discovered

Reconnaissance revealed a **Kubernetes-in-Docker** pattern not fully documented at release time:

```
[Host VM: Ubuntu 22.04]
└── Docker container (OpenShell Gateway)
    └── K3s cluster (Kubernetes lightweight)
        ├── CoreDNS (10.43.0.10)
        ├── Inference proxy (inference.local)
        └── Sandbox pod 'cortana'  ← assessment entry point
            IP: 10.200.0.2/24
            Gateway: 10.200.0.1 (OpenShell proxy → port 3128)
            → NVIDIA Cloud API (internet)
```

### Defense Layers Identified

| Layer | Mechanism | Status | Effectiveness |
|-------|-----------|--------|---------------|
| Filesystem | Landlock LSM (best_effort) | ACTIVE | High |
| Syscalls | seccomp BPF (mode 2, 2 filters) | ACTIVE | High |
| Privileges | Linux Capabilities (CapEff=0) | ACTIVE | High |
| Network egress | CONNECT-only proxy on :3128 | ACTIVE | High |
| TLS inspection | MITM via OpenShell Sandbox CA | ACTIVE | High |
| K3s isolation | Routing table restriction | ACTIVE | High |
| Toolset | Minimal container image | ACTIVE | Medium |
| Credentials | No hardcoded secrets | VERIFIED | High |

### Network Policy Bypass Results (Phase 2)

12 bypass techniques tested — all blocked or mitigated:

| Technique | Result |
|-----------|--------|
| Host header spoofing | BLOCKED |
| IP-based CONNECT | BLOCKED |
| Non-standard ports | BLOCKED |
| Subdomain wildcarding | PARTIAL (objects.githubusercontent.com allowed) |
| SNI mismatch post-CONNECT | MITIGATED (by destination server) |
| Post-CONNECT internal reach | BLOCKED |
| Double CONNECT (nested tunnel) | BLOCKED |
| Non-CONNECT HTTP methods | BLOCKED |
| Data channel via GitHub CDN | OPEN (inherent to GitHub access) |
| mTLS key extraction | BLOCKED (Landlock) |

---

## Residual Attack Surface (Next Phases)

Areas identified for deeper testing:

* **Landlock boundary** — hardlink attacks, symlink traversal, `/proc` data channels, mTLS key extraction attempts
* **Binary allowlist enforcement** — if enforcement uses process name rather than inode/cryptographic verification, substitution attacks may be viable
* **Python3 as recon substitute** — `socket`, `subprocess`, `ctypes`, `os` modules partially replace absent system tools
* **Prompt injection** — full attack surface of OpenClaw inherited; not yet tested
* **Gateway compromise impact** — TLS MITM means gateway compromise exposes all sandbox HTTPS traffic

---

## Test Environment

| Component | Details |
|-----------|---------|
| Host OS | Ubuntu 22.04 LTS (VMware) |
| Docker | Running, socket accessible to sandbox group |
| Node.js | v22.20.1 |
| OpenShell | v0.0.7 (PyPI via uv) |
| NemoClaw | v0.1.0 (cloned from source) |
| Inference | NVIDIA Cloud API — Nemotron 3 Super 120B |
| Sandbox name | cortana |

---

## Repository Structure

```
nemoclaw-security-research/
├── README.md                          ← You are here
├── reports/
│   ├── NemoClaw_Security_Report_Phase1.md    ← Full Phase 1 report
│   ├── NemoClaw_Security_Report_Phase2.md    ← Full Phase 2 report (NEW)
│   └── findings/
│       ├── F-01_k8s_token_mounted.md
│       ├── F-02_k3s_network_isolation.md
│       ├── F-03_pid1_capabilities.md
│       ├── F-04_no_hardcoded_credentials.md
│       ├── F-05_minimal_toolset.md
│       ├── F-06_missing_ssh_handshake_secret.md      (NEW)
│       ├── F-07_gateway_proxy_architecture.md         (NEW)
│       ├── F-08_proxy_bypass_resistance.md            (NEW)
│       ├── F-09_data_channel_githubusercontent.md     (NEW)
│       ├── F-10_tls_mitm_openshell_ca.md              (NEW)
│       ├── F-11_mtls_key_landlock_protected.md        (NEW)
│       └── F-12_non_connect_methods_blocked.md        (NEW)
├── methodology/
│   ├── recon_approach.md              ← Black-box methodology used
│   └── next_phases.md                 ← Planned test cases for Phases 3–4 (UPDATED)
└── DISCLAIMER.md                      ← Responsible disclosure & research scope
```

---

## About This Research

This project is part of a personal portfolio focused on AI security and Red Team research. The assessment was conducted in a self-hosted environment using publicly available software. No production systems were targeted. All testing was performed against a local installation under controlled conditions.

**Researcher:** Mike  
**Handle:** 0xgrimRPR  
**Role:** Junior Cybersecurity Engineer | Red Team Track  
**Active certifications in progress:** CEH Master, OSCP path  
**Research focus:** AI agent security, sandbox escape vectors, autonomous system threat modeling

---

## Disclaimer

This research is conducted for educational and professional development purposes. See [`DISCLAIMER.md`](DISCLAIMER.md) for full responsible disclosure policy and scope statement.

*NemoClaw is a trademark of NVIDIA Corporation. This project is not affiliated with or endorsed by NVIDIA.*
