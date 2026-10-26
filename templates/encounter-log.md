# Encounter log

Add a row whenever a process exposes one of the [playbook](../PLAYBOOK.md) patterns or produces a
reusable fix. Link the authoritative ticket and any procedure written from it.

| Date | Process | Bad habit observed | Correction applied | Durable artifact |
|---|---|---|---|---|
| YYYY-MM-DD | Sequential bare-metal reimage | Frequent manual checks and changing cadence | Switched to measured intervals, sequential completion gates, and end-of-day stop boundary | This playbook; change history |
| YYYY-MM-DD | AI-assisted on-call triage | Agent repeatedly exited before triage because it had no authentication | Added an authentication preflight, retained-evidence requirements, and a tested read-only fallback | [AI triage readiness](../procedures/ai-triage-readiness.md) |
| YYYY-MM-DD | Recurring hardware-event alerts | Incident-only mitigation left the upstream check CRITICAL; initial probes used the wrong receiver | Archived raw SEL with hashes, gated each clear on live health, canaried one host, verified HARD OK and the next natural cycle | [Historical hardware-event alerts](../procedures/historical-hardware-event-alerts.md) |
| YYYY-MM-DD | Metrics forwarder scrape failure | Initial automation conflated failed scrapes with pod failure | Confirmed label cardinality, used an approved deployment-managed restart, verified collectors and real forwarding | [Scrape timeouts from cardinality](../procedures/metrics-cardinality-scrape-timeouts.md) |
| YYYY-MM-DD | Package security snapshots | A zero-byte package and HTTP/2 resets could have been frozen into a snapshot | Repaired only failed packages over HTTP/1.1 with preserved timestamps, required a clean mirror publish, created immutable snapshots | [APT mirror repair](../procedures/apt-mirror-repair-and-immutable-snapshots.md) |
| | | | | |
