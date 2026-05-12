# F-18 — Safety Scripts Read-Only [INFORMATIONAL]

| Property | Value |
|----------|-------|
| ID | F-18 |
| Severity | INFORMATIONAL |
| Phase | 3 (May 2026) |
| Component | Node.js / Sandbox Runtime |

## Description

Node.js safety scripts injected via `NODE_OPTIONS` are mounted read-only. The sandbox process cannot modify these scripts to inject malicious code into Node.js processes.

## Evidence

```
$ ls -la /tmp/nemoclaw-sandbox-safety-net.js
-r--r--r-- 1 root root 5829 [date] nemoclaw-sandbox-safety-net.js

$ ls -la /tmp/nemoclaw-http-proxy-fix.js
-r--r--r-- 1 root root 7401 [date] nemoclaw-http-proxy-fix.js

$ echo "test" >> /tmp/nemoclaw-sandbox-safety-net.js
PermissionError: [Errno 13] Permission denied
```

Scripts are loaded via:
```
NODE_OPTIONS=--require /tmp/nemoclaw-sandbox-safety-net.js --require /tmp/nemoclaw-http-proxy-fix.js
```

## Security Implication

Positive finding. Safety scripts cannot be tampered with from within the sandbox, preserving runtime integrity.
