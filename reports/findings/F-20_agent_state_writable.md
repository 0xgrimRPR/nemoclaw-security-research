# F-20 — Agent State Writable [LOW]

| Property | Value |
|----------|-------|
| ID | F-20 |
| Severity | LOW |
| Phase | 3 (May 2026) |
| Component | OpenClaw Agent Runtime |

## Description

All files under `/sandbox/.openclaw/` are read-write accessible. This includes the agent's memory database, session transcripts, model configuration, and workflow registry. Modification of these files could alter agent behavior.

## Evidence

```
/sandbox/.openclaw/
├── memory/
│   └── main.sqlite          (69KB) [RW] — Agent memory database
├── agents/main/sessions/
│   └── *.jsonl              [RW] — Session transcripts
├── flows/
│   └── registry.sqlite      [RW] — Workflow registry
└── config files              [RW] — Model and plugin configuration
```

All files report `[RW]` permissions for the sandbox user.

## Attack Scenario

1. Modify `main.sqlite` to inject false memories into the agent
2. Alter session transcripts to influence conversation context
3. Change model config to redirect inference or modify behavior
4. Inject workflow entries to trigger unintended actions

## Security Implication

Potential prompt injection vector via file modification. Designated as Phase 4 target for deeper testing.
