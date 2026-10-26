# Production readiness for AI incident triage

Before enabling an AI agent (for example, a CLI coding agent driven by an incident-triage bot) to
investigate incidents for an on-call queue:

1. Have the automation owner provision an approved **non-personal** credential for the agent through the
   automation secret store. Never run the agent on an individual engineer's identity.
2. Run a startup preflight that proves the agent can authenticate **before** it accepts an incident
   assignment.
3. If authentication fails, report an automation-platform failure separately from the infrastructure
   alert and route it to the automation owner. Do not present it as troubleshooting of the affected host.
4. Retain the preflight timestamp, worker identity, CLI exit code, and sanitized error in the automation
   platform's logs.
5. Require a tested read-only fallback path for operations incidents, including the support-host SSH
   route and required remote tooling.

## Completion evidence

A successful headless agent startup on the same worker image and identity used in production, followed
by a synthetic investigation that reaches the read-only diagnostic stage.

## Related

- [Verifying AI-generated root causes](ai-root-cause-verification.md)
- [Empty results are a vantage-point claim](empty-results-vantage-point.md)
