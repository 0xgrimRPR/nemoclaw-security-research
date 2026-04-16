# F-10 — TLS MITM via OpenShell Sandbox CA [MEDIUM]

| Property | Value |
|----------|-------|
| ID | F-10 |
| Severity | MEDIUM |
| Phase | 2 (April 2026) |
| Component | OpenShell Gateway / TLS |

## Description

The OpenShell gateway performs full TLS interception on all HTTPS traffic using a dynamic per-hostname CA.

## Certificate Details

| Property | Value |
|----------|-------|
| Issuer CN | OpenShell Sandbox CA |
| Issuer O | OpenShell |
| Key Algorithm | ECDSA P-256 |
| Validity | 1975-01-01 to 4096-01-01 |
| Cert Size | ~395–431 bytes (dynamic per host) |
| TLS Version | v1.3 |
| Cipher | TLS_AES_256_GCM_SHA384 |

## Trust Injection

```
SSL_CERT_FILE       = /etc/openshell-tls/ca-bundle.pem
REQUESTS_CA_BUNDLE  = /etc/openshell-tls/ca-bundle.pem
CURL_CA_BUNDLE      = /etc/openshell-tls/ca-bundle.pem
NODE_EXTRA_CA_CERTS = /etc/openshell-tls/openshell-ca.pem
```

## Security Implication

Design decision enabling fine-grained policy enforcement. Gateway has full visibility into HTTPS plaintext. Compromise of gateway would expose all sandbox traffic. Detectable by apps using certificate pinning.
