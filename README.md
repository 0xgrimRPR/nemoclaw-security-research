# NemoClaw Security Research

**Black-box security assessment of NVIDIA NemoClaw — AI agent sandbox architecture analysis**

> Conducted within 24 hours of the platform's public release at GTC 2026 (March 16, 2026).  
> This research is complete. All four phases (reconnaissance → prompt injection) have been executed.

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
| Phase 3 | Filesystem / Landlock boundary testing | ✅ Complete |
| Phase 4 | Prompt injection & agent behavior manipulation | ✅ Complete |

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

### Phase 3 — Filesystem / Landlock Boundary (May 2026)

Ten findings documented across filesystem access, symlink/hardlink attacks, procfs analysis, and privilege controls:

| ID | Title | Severity |
|----|-------|----------|
| F-13 | Root directory not listable (Landlock) | INFORMATIONAL (positive) |
| F-14 | /proc/self/environ readable but no secrets | INFORMATIONAL (positive) |
| F-15 | Symlink traversal blocked by Landlock | INFORMATIONAL (positive) |
| F-16 | Hardlink attacks blocked (cross-device + kernel) | INFORMATIONAL (positive) |
| F-17 | PID 1 cmdline/status readable (info leak) | LOW |
| F-18 | NODE_OPTIONS safety scripts read-only | INFORMATIONAL (positive) |
| F-19 | DNS changed to direct proxy at 10.200.0.1 | INFORMATIONAL |
| F-20 | Agent state writable (/sandbox/.openclaw/) | LOW |
| F-21 | Seccomp increased to 3 BPF filters | INFORMATIONAL (positive) |
| F-22 | NoNewPrivs=1 enforced | INFORMATIONAL (positive) |

Full details in [`/reports/NemoClaw_Security_Report_Phase3.md`](/reports/NemoClaw_Security_Report_Phase3.md).

### Phase 4 — Prompt Injection & Agent Behavior Manipulation (May 2026)

Six findings documented across prompt injection testing, memory poisoning, and egress exfiltration:

| ID | Title | Severity |
|----|-------|----------|
| F-23 | Indirect prompt injection via docstring → credential disclosure | HIGH |
| F-24 | Operator token + private key accessible within sandbox | MEDIUM |
| F-25 | Device authentication disabled by default | MEDIUM |
| F-26 | Agent detects injection but completes task first | LOW |
| F-27 | Session-level memory poisoning → autonomous credential disclosure | HIGH |
| F-28 | Agent prepares exfiltration payload for allowed egress endpoint | HIGH |

Full details in [`/reports/NemoClaw_Security_Report_Phase4.md`](/reports/NemoClaw_Security_Report_Phase4.md).

---

## Architecture Discovered

Reconnaissance revealed a **Kubernetes-in-Docker** pattern not fully documented at release time:

```
[Host VM: Ubuntu 22.04]
└── Docker container (OpenShell Gateway)
    └── K3s cluster (Kubernetes lightweight)
        ├── DNS proxy (10.200.0.1)
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
| Syscalls | seccomp BPF (mode 2, 3 filters) | ACTIVE | High |
| Privileges | Linux Capabilities (CapEff=0) + NoNewPrivs | ACTIVE | High |
| Network egress | CONNECT-only proxy on :3128 | ACTIVE | High |
| TLS inspection | MITM via OpenShell Sandbox CA | ACTIVE | High |
| DNS | Direct proxy (10.200.0.1) | ACTIVE | High |
| K3s isolation | Routing table restriction | ACTIVE | High |
| Toolset | Minimal container image | ACTIVE | Medium |
| Credentials | No hardcoded secrets | VERIFIED | High |
| Runtime safety | Node.js scripts via NODE_OPTIONS (read-only) | ACTIVE | High |

### Key Assessment Results by Phase

**Phase 2 — Network Policy Bypass:** 12 bypass techniques tested, all blocked or mitigated. One data channel identified via GitHub CDN (F-09).

**Phase 3 — Filesystem Boundary:** Symlink traversal, hardlink attacks, and procfs enumeration all blocked by Landlock. Agent state files writable (F-20) — confirmed exploitable in Phase 4.

**Phase 4 — Prompt Injection:** 6 tests executed. Agent blocked obvious injection (T-01, T-02) but compromised by contextual injection (T-03), direct memory poisoning (T-04/T-05), and egress exfiltration prep (T-06). Three HIGH findings. Complete attack chain demonstrated from planted file to exfiltration-ready payload.

---

## Project-Wide Conclusions

### What NemoClaw Gets Right

NemoClaw demonstrates one of the most thoughtful approaches to AI agent sandboxing in 2026. Six independent infrastructure layers create meaningful barriers to sandbox escape. Phases 1–3 confirmed all infrastructure controls function as intended.

### The Fundamental Gap

Despite robust infrastructure, a critical gap exists between **protecting the host from the agent** and **protecting the agent's data from the agent itself**. The agent has legitimate access to credentials, can be manipulated into disclosing them, and can prepare exfiltration payloads targeting allowed egress endpoints. No security control at any layer addresses this path.

This is not NemoClaw-specific — it is a structural challenge for all AI agent platforms.

---

## Test Environment

| Component | Details |
|-----------|---------|
| Host OS | Ubuntu 22.04 LTS (VMware) |
| Docker | Running, socket accessible to sandbox group |
| Node.js | v22.20.1 |
| OpenShell | v0.0.36 (PyPI via uv) |
| NemoClaw | v0.0.38 / v0.1.0 |
| OpenClaw | 2026.4.24 (cbcfdf6) |
| Inference | NVIDIA Cloud API — Nemotron 3 Super 120B |
| Sandbox name | cortana |

---

## Repository Structure

```
nemoclaw-security-research/
├── README.md                          ← You are here
├── reports/
│   ├── NemoClaw_Security_Report_Phase1.md
│   ├── NemoClaw_Security_Report_Phase2.md
│   ├── NemoClaw_Security_Report_Phase3.md
│   ├── NemoClaw_Security_Report_Phase4.md           (NEW)
│   └── findings/
│       ├── F-01 through F-22                        (Phases 1–3)
│       ├── F-23_prompt_injection_credential_disclosure.md    (NEW)
│       ├── F-24_operator_token_accessible.md                 (NEW)
│       ├── F-25_device_auth_disabled.md                      (NEW)
│       ├── F-26_injection_detected_task_completed.md         (NEW)
│       ├── F-27_session_memory_poisoning.md                  (NEW)
│       └── F-28_exfiltration_payload_prepared.md             (NEW)
├── methodology/
│   ├── recon_approach.md
│   └── next_phases.md
└── DISCLAIMER.md
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
