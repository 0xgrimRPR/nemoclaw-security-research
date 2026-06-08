# Responsible Disclosure & Research Scope

## Scope Statement

This research was conducted against a **local, self-hosted installation** of NVIDIA NemoClaw (0.0.x early preview), running in a dedicated VMware virtual machine under the researcher's full control.

- No production systems were targeted
- No NVIDIA infrastructure was accessed beyond the intended public NVIDIA Cloud API endpoint (used exactly as documented for model inference)
- No credentials, tokens, or access belonging to third parties were accessed or retained. Credential values observed inside the local sandbox (the researcher's own throwaway instance) are redacted in published evidence.
- All testing was performed in an isolated local environment

## Intent

The goal of this research is to understand the security architecture of an emerging AI agent platform from the perspective of an external assessor — specifically to:

1. Document the actual (not just intended) defense layers in place at release
2. Identify residual attack surface for responsible communication to the vendor and the security community
3. Contribute to the body of practical knowledge around AI agent sandbox security

## Findings Disposition

Findings in this repository span the full severity range:

- **Informational / positive** — controls working as intended (the majority of Phases 1–3)
- **Low** — mitigated by existing controls, with a recommendation to monitor (e.g., F-01, F-09, F-17, F-20, F-26)
- **Medium** — F-10 (TLS MITM design trade-off), F-24, F-25
- **High** — F-23, F-27, F-28 (Phase 4: prompt-injection-driven credential disclosure and preparation of an exfiltration payload)

The Medium and High findings concern a structural class of AI-agent risk — the agent's own legitimate access to the credentials it needs to operate — that is **not unique to NemoClaw** and has been documented publicly by independent researchers (e.g., Lasso Security, May 2026).

## Disclosure

This research was performed against a local, self-hosted, open-source preview and targeted no third-party or production systems. Operators evaluating NemoClaw are encouraged to review the Phase 4 findings and the recommendations therein.

For coordinated reporting of the Medium/High findings, NVIDIA's vulnerability disclosure program is available at: https://www.nvidia.com/en-us/security/

> **Maintainer note:** confirm the coordinated-disclosure status with NVIDIA PSIRT for the Medium/High findings and record it here before relying on this section.

## Legal

This research is conducted for educational and professional development purposes under the principle of good-faith security research. The researcher did not exceed authorized access, cause damage to any system, or obtain any information beyond what was necessary to document the security architecture.

*NemoClaw and OpenShell are products of NVIDIA Corporation. This project is not affiliated with or endorsed by NVIDIA.*
