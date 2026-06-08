# NemoClaw Security Assessment — Phase 2 Report

**Gateway Analysis & Network Policy Bypass Testing**

| Field | Value |
|-------|-------|
| Date | April 16, 2026 |
| Analyst | 0xgrimRPR |
| Role | Junior Cybersecurity Engineer / Red Team Track |
| Target | NVIDIA NemoClaw 0.0.x (early preview) + OpenShell 0.0.7 |
| Environment | VMware VM / Ubuntu 22.04 LTS / No GPU |
| Scope | Gateway analysis & network policy bypass |
| Classification | Research Use Only |
| Previous Report | NemoClaw_Security_Report_Phase1.md (March 17, 2026) |

---

## 1. Executive Summary

This report documents two parts of the overall Phase 2 assessment — Part A (Gateway Analysis) and Part B (Network Policy Bypass) — continuing the security reconnaissance initiated in Report #1 (March 17, 2026). Testing was conducted on April 16, 2026, approximately one month after the initial assessment.

During environment setup, a reproducible bug was discovered in OpenShell 0.0.7: the onboarding and gateway provisioning processes fail to create a required Kubernetes secret (`openshell-ssh-handshake`), preventing sandbox initialization. This bug was manually remediated to proceed with testing.

Gateway analysis revealed a forward HTTP proxy operating on port 3128 within the sandbox network namespace. The proxy enforces network policies via CONNECT tunnel validation with hostname:port exact matching. All non-CONNECT HTTP methods are rejected. A TLS Man-in-the-Middle (MITM) architecture was discovered: the OpenShell Sandbox CA intercepts all HTTPS traffic, generating dynamic certificates per hostname. This enables full traffic visibility at the gateway layer.

Network policy bypass testing confirmed that the proxy is resistant to common evasion techniques including Host header spoofing, IP-based CONNECT, port smuggling, and subdomain wildcarding. One data channel was identified via `objects.githubusercontent.com`, which is permitted alongside `github.com` and `api.github.com` in the default policy configuration.

**Seven new findings were documented (F-06 through F-12)**, including one **BUG** (missing K8s secret), one **MEDIUM** (TLS MITM architecture), one **LOW** (data channel), and four **INFORMATIONAL** findings confirming the effectiveness of the proxy security controls.

---

## 2. Environment Changes Since Report #1

| Component | Report #1 (March) | Report #2 (April) |
|-----------|-------------------|-------------------|
| seccomp filters | 1 BPF filter | 2 BPF filters (additional filter added) |
| Gateway port | Not documented | 3128 (forward proxy) |
| Active policies | pypi, npm + defaults | github only (sandbox recreated) |
| Secret provisioning | Worked (first install) | Bug: openshell-ssh-handshake missing |
| Network interface | veth-s-719f5913 | veth-s-c467b47b (new sandbox) |

---

## 3. Gateway Analysis (Part A)

### 3.1 Gateway Architecture Discovery

The gateway at `10.200.0.1` was determined to be a forward HTTP proxy listening on port 3128. This was not discoverable via conventional port scanning (ports 1–1024 plus common high ports returned no results) because the sandbox communicates with the proxy exclusively through environment-configured proxy variables, not direct TCP connections to visible ports.

### 3.2 Proxy Configuration

The following environment variables route all sandbox traffic through the proxy:

```
http_proxy  = http://10.200.0.1:3128
https_proxy = http://10.200.0.1:3128
HTTP_PROXY  = http://10.200.0.1:3128
HTTPS_PROXY = http://10.200.0.1:3128
ALL_PROXY   = http://10.200.0.1:3128
no_proxy    = 127.0.0.1,localhost,::1
```

### 3.3 Proxy Behavior

- **Method restriction:** Only CONNECT method is permitted. GET, POST, PUT, and DELETE via proxy return `403 Forbidden`.
- **Policy enforcement:** CONNECT requests are validated against a policy whitelist using exact `hostname:port` matching. Denied requests return structured JSON responses with error details.
- **DNS resolution:** DNS is non-functional from the sandbox. CoreDNS at `10.43.0.10` is unreachable (no route to K3s network). The proxy resolves hostnames on behalf of the sandbox after accepting a CONNECT request.

### 3.4 Policy Deny Response Format

```
HTTP/1.1 403 Forbidden
Content-Type: application/json

{"detail":"CONNECT <host>:<port> not permitted by policy",
 "error":"policy_denied"}
```

### 3.5 Active Network Policies

Testing via raw socket CONNECT revealed the following policy state on the recreated sandbox:

| Endpoint | Status | Notes |
|----------|--------|-------|
| `github.com:443` | ✅ ALLOW | Default policy |
| `api.github.com:443` | ✅ ALLOW | Default policy |
| `objects.githubusercontent.com:443` | ✅ ALLOW | GitHub CDN for binary assets |
| `integrate.api.nvidia.com:443` | ❌ DENY | Policy not applied post-recreate |
| `inference-api.nvidia.com:443` | ❌ DENY | Policy not applied post-recreate |
| `clawhub.com:443` | ❌ DENY | Policy not applied post-recreate |
| `openclaw.ai:443` | ❌ DENY | Policy not applied post-recreate |
| `api.anthropic.com:443` | ❌ DENY | claude_code preset not applied |
| `registry.npmjs.org:443` | ❌ DENY | npm preset marked active but not enforced |
| `api.telegram.org:443` | ❌ DENY | telegram preset not applied |
| `google.com:443` | ❌ DENY | Not in any policy |

> **Nota / Note:** La discrepancia entre las policies marcadas como activas (`npm ●`, `pypi ●`) y el comportamiento real del proxy sugiere un bug en la sincronización de policies al recrear el gateway. / The discrepancy between policies marked as active and actual proxy behavior suggests a bug in policy synchronization when the gateway is recreated.

---

## 4. TLS Man-in-the-Middle Architecture

### 4.1 Discovery

Analysis of TLS certificates presented during CONNECT tunnels revealed that the OpenShell gateway performs full TLS interception on all HTTPS traffic transiting the proxy.

### 4.2 MITM Certificate Details

| Property | Value |
|----------|-------|
| Issuer CN | OpenShell Sandbox CA |
| Issuer O | OpenShell |
| Key Algorithm | ECDSA P-256 (secp256r1) |
| Validity Start | 1975-01-01T00:00:00Z |
| Validity End | 4096-01-01T00:00:00Z |
| Certificate Size | ~395–431 bytes (varies per hostname) |
| TLS Version | TLSv1.3 |
| Cipher Suite | TLS_AES_256_GCM_SHA384 |
| Generation | Dynamic per-hostname (CN matches target) |

### 4.3 Trust Injection Mechanism

The OpenShell CA is trusted by sandbox processes via environment variables, **not** via the system trust store:

```
SSL_CERT_FILE       = /etc/openshell-tls/ca-bundle.pem    (147 certs, 220KB)
REQUESTS_CA_BUNDLE  = /etc/openshell-tls/ca-bundle.pem
CURL_CA_BUNDLE      = /etc/openshell-tls/ca-bundle.pem
NODE_EXTRA_CA_CERTS = /etc/openshell-tls/openshell-ca.pem  (1 cert, 644B)
```

The `/etc/openshell-tls/` directory also contains a `client/` subdirectory with an mTLS private key (`tls.key`, 241 bytes) that is protected by Landlock (read access denied to sandbox user). This key is used for sandbox-to-gateway mutual TLS authentication.

### 4.4 Security Implications

The TLS MITM architecture means the gateway has full visibility into all HTTPS request and response bodies, including authentication tokens, API keys, and data payloads. This is a **design decision** (not a vulnerability) that enables the policy engine to enforce fine-grained controls beyond hostname:port matching. However, it means that any compromise of the gateway process would expose all sandbox HTTPS traffic in plaintext.

---

## 5. Network Policy Bypass Testing (Part B)

Twelve bypass techniques were tested against the proxy policy engine. Ten were blocked or mitigated; two endpoints (the objects.githubusercontent.com data channel and the GitHub API) were permitted by policy and are tracked separately (F-09). No policy-restricted destination was reached.

| # | Technique | Result | Details |
|---|-----------|--------|---------|
| 1 | HTTP redirect following | BLOCKED | CONNECT to github.com succeeds; post-tunnel redirects handled by destination server, not proxy |
| 2 | Host header spoofing | BLOCKED | CONNECT google.com with Host: github.com returns 403; proxy validates CONNECT target, not Host header |
| 3 | IP-based CONNECT | BLOCKED | CONNECT via raw IP (140.82.121.4) returns 403; proxy requires hostname, IPs rejected |
| 4 | Non-standard ports | BLOCKED | github.com on ports 80/8080/8443/22/9090 all return 403; policy is hostname:443 exact match |
| 5 | Subdomain wildcarding | PARTIAL | gist.github.com, raw.githubusercontent.com, pages.github.com all denied; `objects.githubusercontent.com` allowed |
| 6 | SNI mismatch post-CONNECT | MITIGATED | CONNECT github.com + TLS SNI=google.com: GitHub returns 301; proxy does not inspect post-CONNECT TLS but destination rejects mismatched SNI |
| 7 | Data channel via objects CDN | OPEN | objects.githubusercontent.com:443 CONNECT succeeds; Varnish CDN responds; potential data exfil channel |
| 8 | Post-CONNECT internal reach | BLOCKED | Raw HTTP to 10.43.0.1 (K8s API) via tunnel produces empty response; tunnel goes direct to internet |
| 9 | Double CONNECT (nested) | BLOCKED | Second CONNECT inside established tunnel produces empty response; GitHub is not a proxy |
| 10 | Non-CONNECT methods | BLOCKED | Plain HTTP GET/POST/PUT/DELETE through proxy all return 403; only CONNECT permitted |
| 11 | GitHub API access | ALLOWED | api.github.com CONNECT succeeds; API reachable (403 = rate limit, no auth token) |
| 12 | mTLS key extraction | BLOCKED | Landlock prevents reading /etc/openshell-tls/client/tls.key |

---

## 6. Security Findings

Findings F-06 through F-12 continue the numbering from Report #1 (F-01 through F-05).

---

### F-06 — OpenShell 0.0.7 Missing K8s Secret During Onboard [BUG]

**Description:** The OpenShell gateway provisioning process (`gateway start`) and NemoClaw onboarding (`nemoclaw onboard`) fail to create the Kubernetes secret `openshell-ssh-handshake` required by the OpenShell Helm chart. The pod `openshell-0` enters `CreateContainerConfigError` state and loops indefinitely. This bug is **reproducible**: destroying and recreating the gateway triggers the same failure.

**Evidence:**

```
$ docker logs openshell-cluster-openshell --tail 5
E0416 pod_workers.go:1324] "Error syncing pod, skipping"
  err="CreateContainerConfigError: secret
  \"openshell-ssh-handshake\" not found"

# Manual fix:
$ kubectl create secret generic openshell-ssh-handshake \
    --from-literal=secret="$(openssl rand -hex 32)" \
    -n openshell
```

**Security Implication:** Sandbox cannot start without manual intervention. New users would be unable to use NemoClaw after initial setup. This is a blocking defect for the onboarding experience.

---

### F-07 — Gateway Proxy Architecture on Port 3128 [INFORMATIONAL]

**Description:** The OpenShell gateway operates as a forward HTTP proxy on port 3128 within the sandbox network namespace. The proxy only accepts CONNECT method requests, validates destinations against a policy whitelist using exact `hostname:port` matching, and returns structured JSON error responses for denied requests.

**Evidence:**

```
$ env | grep -i proxy
http_proxy=http://10.200.0.1:3128
https_proxy=http://10.200.0.1:3128

# CONNECT denied response:
{"detail":"CONNECT host:port not permitted by policy",
 "error":"policy_denied"}
```

**Security Implication:** Positive finding. The proxy architecture provides a clean enforcement point for network policy. The CONNECT-only restriction limits the attack surface to tunnel-based communication.

---

### F-08 — Proxy Resistant to Common Bypass Techniques [INFORMATIONAL]

**Description:** The proxy policy engine was tested against twelve bypass techniques (Host header spoofing, IP-based CONNECT, non-standard ports, subdomain wildcarding, SNI mismatch, post-CONNECT internal reach, nested CONNECT, non-CONNECT methods, and more). Ten were blocked or mitigated; two endpoints were permitted by policy and are tracked separately (F-09).

**Evidence:**

```
CONNECT google.com:443 Host: github.com  → 403 Forbidden
CONNECT 140.82.121.4:443                 → 403 Forbidden
CONNECT github.com:80                    → 403 Forbidden
CONNECT gist.github.com:443             → 403 Forbidden
GET http://github.com/                   → 403 Forbidden
```

**Security Implication:** Positive finding. The policy engine demonstrates robust enforcement. Hostname-based matching with port restriction and method limitation creates a narrow, well-defined egress surface.

---

### F-09 — Data Channel via objects.githubusercontent.com [LOW]

**Description:** The GitHub policy permits CONNECT to `objects.githubusercontent.com:443`, which serves binary assets (release downloads, upload artifacts) via Varnish CDN. An attacker with write access to a GitHub repository could use this endpoint as a bidirectional data channel.

**Evidence:**

```
CONNECT objects.githubusercontent.com:443 → 200 Connection Established
GET / → HTTP/1.1 404 Not Found
Server: Varnish
X-Served-By: cache-iah-kiah1900045-IAH
```

**Security Implication:** The GitHub policy implicitly permits a data channel that could be used for payload delivery or data exfiltration. This is inherent to allowing GitHub access and is mitigated by the TLS MITM visibility. Consider monitoring for unusual download patterns.

---

### F-10 — TLS MITM via OpenShell Sandbox CA [MEDIUM]

**Description:** The OpenShell gateway performs full TLS interception on all HTTPS traffic. Dynamic certificates are generated per-hostname using an internal ECDSA P-256 CA (`CN=OpenShell Sandbox CA, O=OpenShell`). The CA is injected via environment variables (`SSL_CERT_FILE`, `REQUESTS_CA_BUNDLE`, `CURL_CA_BUNDLE`, `NODE_EXTRA_CA_CERTS`). The system trust store is not modified.

**Evidence:**

```
Cert Issuer:  CN=OpenShell Sandbox CA, O=OpenShell
Cert Subject: CN=github.com (dynamic per host)
Validity:     1975-01-01 to 4096-01-01
Key:          ECDSA P-256 | TLS: v1.3 | Cipher: AES-256-GCM-SHA384

CA location:  /etc/openshell-tls/openshell-ca.pem (644 bytes)
Bundle:       /etc/openshell-tls/ca-bundle.pem (147 certs, 220KB)
```

**Security Implication:** The gateway has full visibility into all HTTPS plaintext including authentication tokens, API keys, and data payloads. This is an architectural design choice enabling fine-grained policy enforcement. However, compromise of the gateway process would expose all sandbox traffic. The MITM is transparent to applications respecting the environment variables but could be detected by applications using certificate pinning.

---

### F-11 — mTLS Client Key Protected by Landlock [INFORMATIONAL]

**Description:** The sandbox-to-gateway mutual TLS client key is stored at `/etc/openshell-tls/client/tls.key` (241 bytes, ECDSA). Landlock prevents the sandbox user from reading this file.

**Evidence:**

```
$ cat /etc/openshell-tls/client/tls.key
Permission denied
```

**Security Implication:** Positive finding. The mTLS key is correctly protected. Extraction would require a Landlock bypass (tested in Phase 3).

---

### F-12 — Non-CONNECT HTTP Methods Blocked at Proxy [INFORMATIONAL]

**Description:** The proxy rejects all non-CONNECT HTTP methods (GET, POST, PUT, DELETE) with `403 Forbidden`. This prevents direct HTTP requests through the proxy and limits sandbox egress to CONNECT tunnels only.

**Evidence:**

```
GET http://github.com/ HTTP/1.1    → 403 Forbidden
POST http://github.com/ HTTP/1.1   → 403 Forbidden
PUT http://github.com/ HTTP/1.1    → 403 Forbidden
DELETE http://github.com/ HTTP/1.1 → 403 Forbidden
```

**Security Implication:** Positive finding. CONNECT-only restriction prevents plain HTTP proxy abuse and ensures all egress goes through tunnel-based TLS connections subject to the MITM inspection.

---

## 7. Cumulative Findings Summary

| ID | Finding | Severity | Report |
|----|---------|----------|--------|
| F-01 | K8s Service Account Token Mounted but Inaccessible | LOW | #1 (March) |
| F-02 | K3s Internal Network Unreachable from Sandbox | INFO | #1 (March) |
| F-03 | PID 1 Runs as Root with Non-Zero Capabilities | INFO | #1 (March) |
| F-04 | No Hardcoded Credentials in Distributed Code | INFO | #1 (March) |
| F-05 | Minimal Toolset Limits Post-Exploitation | INFO | #1 (March) |
| F-06 | OpenShell 0.0.7 Missing K8s Secret During Onboard | BUG | #2 (April) |
| F-07 | Gateway Proxy Architecture on Port 3128 | INFO | #2 (April) |
| F-08 | Proxy Resistant to Common Bypass Techniques | INFO | #2 (April) |
| F-09 | Data Channel via objects.githubusercontent.com | LOW | #2 (April) |
| F-10 | TLS MITM via OpenShell Sandbox CA | MEDIUM | #2 (April) |
| F-11 | mTLS Client Key Protected by Landlock | INFO | #2 (April) |
| F-12 | Non-CONNECT HTTP Methods Blocked at Proxy | INFO | #2 (April) |

---

## 8. Next Steps — Remaining Phases

### Phase 3 — Filesystem / Landlock Testing

**Status:** Not started. Planned for next session.

- Enumerate all readable paths via Python3 full-path survey
- Symlink traversal from `/tmp` toward restricted paths
- Hardlink attempt against read-only paths (Landlock v1 limitation)
- `/proc` as alternative data channel (`/proc/1/environ`, `/proc/1/maps`, `/proc/1/cmdline`)
- Attempt to read the mTLS client key via indirect methods

### Phase 4 — Prompt Injection Testing

**Status:** Not started. Planned for future session.

- Direct system prompt override attempts
- Indirect injection via agent-readable files in `/sandbox`
- Agent-initiated requests to non-whitelisted hosts
- System prompt extraction via conversational manipulation

---

## 9. Conclusions

Part A and Part B testing confirms that the OpenShell gateway implements a well-designed network security boundary. The CONNECT-only proxy with hostname:port exact matching, combined with TLS MITM inspection, creates a comprehensive traffic control layer that resisted all tested bypass techniques.

The most significant finding is the TLS MITM architecture (F-10), which is a deliberate design choice rather than a vulnerability. It provides the gateway with full traffic visibility, enabling future fine-grained controls (e.g., HTTP method restrictions per endpoint, request body inspection). However, it also means the gateway is a high-value target — its compromise would expose all sandbox HTTPS traffic.

The OpenShell 0.0.7 onboarding bug (F-06) is the only blocking defect found. It prevents sandbox initialization entirely and requires manual Kubernetes secret creation to resolve. This should be reported upstream.

Overall, the network security controls continue to demonstrate maturity beyond what is typical for alpha-stage software.

---

*Report generated: April 16, 2026 | NemoClaw 0.0.x (early preview) + OpenShell 0.0.7 | For research purposes only*

*Nota: Este reporte es continuación de NemoClaw_Security_Report_Phase1.md. Las fases 3 y 4 se documentarán en reportes subsecuentes.*
