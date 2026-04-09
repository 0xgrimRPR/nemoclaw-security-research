# Planned Testing — Phases 2–4

This document outlines the test cases planned for the next research sessions, based on the residual attack surface identified in Phase 1.

---

## Phase 2 — Gateway Analysis

The OpenShell gateway at `10.200.0.1` is the single point of control for all sandbox egress. It handles inference interception, network policy enforcement, and egress proxying. A vulnerability here could allow policy bypass without touching the sandbox itself.

**Test cases:**

- Port scan of `10.200.0.1` via Python3 socket to enumerate exposed services
- Enumerate admin/health/metrics endpoints on the gateway (common paths: `/health`, `/metrics`, `/debug`, `/admin`)
- Analyze headers added by the proxy to inference requests (potential info leakage)
- Test whether the gateway validates the `Host` header or is vulnerable to host header injection

---

## Phase 3 — Network Policy Bypass

The network whitelist allows a defined set of endpoints by binary path and domain. The question is whether the enforcement mechanism can be circumvented.

**Test cases:**

- HTTP redirect following: does the gateway follow redirects to non-whitelisted hosts?
- DNS rebinding via Python3 socket: resolve a controlled domain that returns a whitelisted IP
- CONNECT tunnel attempt through a whitelisted endpoint (e.g., using `github.com` as a pivot)
- TLS certificate validation behavior: does the proxy validate certificates, or can a MITM position be established?
- Binary allowlist bypass: can a symlink or renamed binary pass the binary path check?

---

## Phase 4 — Filesystem / Landlock Boundary Testing

Landlock is configured in `best_effort` mode. This means if the kernel doesn't fully support a requested restriction, it silently degrades rather than failing closed. The goal is to probe the actual boundaries.

**Test cases:**

- Full readable path enumeration via Python3 (map what the sandbox user can actually read)
- Symlink traversal in `/tmp` toward restricted paths
- Hardlink attempt against read-only paths (Landlock v1 does not restrict hardlinks in all configurations)
- `/proc` as alternative data channel: can `/proc/1/environ`, `/proc/1/maps`, or `/proc/1/fd/*` yield useful information?
- Test whether `/opt/nemoclaw/dist/` contains anything sensitive in newer versions

---

## Phase 5 — Prompt Injection & Agent Behavior

This phase targets NemoClaw as an AI agent platform, not just as a sandbox. The attack surface is the agent's reasoning and instruction-following behavior.

**Test cases:**

- Direct system prompt override attempts via user-controlled input
- Indirect injection via agent-readable files in `/sandbox` (e.g., a malicious README the agent reads)
- Agent-initiated requests to non-whitelisted hosts via tool calls
- System prompt extraction: can the agent be induced to reveal its instructions?
- Chain-of-thought manipulation: can attacker input influence intermediate reasoning steps?
- Tool call injection: if the agent can execute code, can injected input change what it executes?

---

*Updated: March 2026 | Based on Phase 1 findings*
