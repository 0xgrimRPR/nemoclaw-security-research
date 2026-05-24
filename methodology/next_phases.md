# Next Phases — Planned Test Cases

> Updated: May 22, 2026 (post-Phase 4 completion)
> All planned phases are now complete.

---

## Completed Phases

### Phase 1 — Architecture Mapping & Defense Layer Enumeration ✅

Completed March 17, 2026. See `reports/NemoClaw_Security_Report_Phase1.md`.

Five findings (F-01 through F-05) documented. Kubernetes-in-Docker architecture mapped. All five defense layers verified as active.

### Phase 2 — Gateway Analysis & Network Policy Bypass ✅

Completed April 16, 2026. See `reports/NemoClaw_Security_Report_Phase2.md`.

Seven findings (F-06 through F-12) documented. Gateway proxy architecture on port 3128 discovered. TLS MITM via OpenShell Sandbox CA confirmed. All bypass techniques blocked or mitigated. One blocking onboarding bug (F-06) identified.

### Phase 3 — Filesystem / Landlock Boundary Testing ✅

Completed May 9, 2026. See `reports/NemoClaw_Security_Report_Phase3.md`.

Ten findings (F-13 through F-22) documented. Landlock LSM confirmed comprehensive filesystem boundary enforcement. Root directory not listable, symlink/hardlink attacks blocked, mTLS key protected, NoNewPrivs enforced. Agent state files writable (F-20) identified as Phase 4 target.

### Phase 4 — Prompt Injection & Agent Behavior Manipulation ✅

Completed May 22, 2026. See `reports/NemoClaw_Security_Report_Phase4.md`.

Six findings (F-23 through F-28) documented. Six prompt injection tests executed: 2 blocked, 4 successful. Demonstrated complete attack chain from planted file → credential disclosure → session contamination → exfiltration payload preparation. Three HIGH severity findings. Confirmed that infrastructure controls protect the host but do not protect the agent's data from the agent itself.

---

## Potential Future Research Directions

The following areas were not tested but could extend this research:

- **Model comparison:** Test prompt injection resistance across Nemotron Ultra 253B, Nemotron Super 49B, and Nemotron Nano 30B to determine if model size affects susceptibility
- **Emoji/encoding-based exfiltration:** Replicate Lasso Security's emoji-encoding technique to test if network policy static filters can be evaded
- **SQLite memory write-path:** Determine if the agent can be manipulated into writing malicious entries to `main.sqlite` for cross-session persistent poisoning
- **Workflow registry injection:** Modify `registry.sqlite` (F-20 confirmed writable) to trigger unintended agent actions
- **Multi-agent scenarios:** Test if a compromised agent session in one sandbox can influence agents in other sandboxes via shared workspace files

---

*Last updated: May 22, 2026 — All four assessment phases complete*
