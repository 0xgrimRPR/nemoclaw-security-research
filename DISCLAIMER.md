# Responsible Disclosure & Research Scope

## Scope Statement

This research was conducted against a **local, self-hosted installation** of NVIDIA NemoClaw v0.1.0, running in a dedicated VMware virtual machine under the researcher's full control.

- No production systems were targeted
- No NVIDIA infrastructure was accessed beyond the intended public NVIDIA Cloud API endpoint (used exactly as documented for model inference)
- No credentials, tokens, or access belonging to third parties were accessed or retained
- All testing was performed in an isolated local environment

## Intent

The goal of this research is to understand the security architecture of an emerging AI agent platform from the perspective of an external assessor — specifically to:

1. Document the actual (not just intended) defense layers in place at release
2. Identify residual attack surface for responsible communication to the vendor and the security community
3. Contribute to the body of practical knowledge around AI agent sandbox security

## Findings Disposition

All findings in this repository are either:

- **Informational / positive findings** — documenting controls that work as intended
- **Low severity** — mitigated by existing controls, with a recommendation to monitor

No critical or high-severity vulnerabilities were identified in Phase 1. If critical findings are discovered in future phases, they will be reported to NVIDIA's security team via their responsible disclosure process before public disclosure.

## NVIDIA Security Contact

NVIDIA's vulnerability disclosure program: https://www.nvidia.com/en-us/security/

## Legal

This research is conducted for educational and professional development purposes under the principle of good-faith security research. The researcher did not exceed authorized access, cause damage to any system, or obtain any information beyond what was necessary to document the security architecture.

*NemoClaw and OpenShell are products of NVIDIA Corporation. This project is not affiliated with or endorsed by NVIDIA.*
