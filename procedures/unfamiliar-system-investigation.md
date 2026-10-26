# Unfamiliar-system investigation and incident causality

## Discovery sequence (AI-assisted)

Use this exact sequence before touching a system you do not own:

1. Ask the AI assistant to search work items, chat, and email for the service name, all hostnames/device
   IDs, the parent ticket, and the terms `owner`, `runbook`, `on-call`, `decommission`, and `rollback`.
2. Ask it to search the relevant repositories and wiki for service definitions, inventory, deployment
   code, monitoring configuration, `CODEOWNERS`, and escalation aliases.
3. Produce an architecture table listing component, purpose, upstream dependencies, downstream consumers,
   owner evidence, and shutdown consequence.
4. Produce an owner table listing person/team, source link, evidence date, requested approval, and
   response status.
5. Ask the identified service owner to confirm consumers and impact; ask the platform owner to confirm the
   procedure; ask each dependency owner to approve removal or state prerequisites.
6. Put those confirmations in the authoritative change record before destructive execution.

The AI does the searching and tabulating; humans confirm ownership and approve. Label every AI-sourced
owner as **inferred** until a person confirms it.

## Incident causality

- Capture a baseline before remediation.
- Correlate timestamps between actions and symptoms.
- Test whether the failure exists outside the changed component.
- Avoid declaring causality until competing explanations are checked.
