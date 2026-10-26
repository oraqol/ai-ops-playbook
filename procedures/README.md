# Concrete procedures

Each procedure applies the [playbook](../PLAYBOOK.md) rules to a real failure class: detection criteria,
authorized remediation, verification that proves recovery, and conditions where the fix must not be used.
Identifiers from the original environment have been replaced with generic placeholders.

## AI-assisted operations

| Procedure | Use when |
|---|---|
| [Verifying AI-generated root causes](ai-root-cause-verification.md) | An AI SRE agent or triage bot reports a cause |
| [Production readiness for AI incident triage](ai-triage-readiness.md) | Enabling an AI agent on an on-call queue |
| [Unfamiliar-system investigation](unfamiliar-system-investigation.md) | Using AI to map ownership and dependencies before change |
| [Verifying configuration-management deployments](config-management-deploy-verification.md) | A change (including an AI-agent branch) "should be live" |
| [Empty results are a vantage-point claim](empty-results-vantage-point.md) | Any "not found" or "unreachable" result |

## Infrastructure

| Procedure | Use when |
|---|---|
| [APT mirror repair and immutable snapshotting](apt-mirror-repair-and-immutable-snapshots.md) | A mirrored package is zero-byte or checksum-invalid |
| [Bare-metal wipe and reimage gates](bare-metal-reimage-gates.md) | Sequential reimage or retirement of physical hosts |
| [Stalled Kubernetes drains](kubernetes-node-drain-stalls.md) | Automated cordon-and-drain does not finish |
| [Metrics scrape timeouts from label cardinality](metrics-cardinality-scrape-timeouts.md) | Scrapes fail while pods look healthy |
| [Historical hardware-event alerts](historical-hardware-event-alerts.md) | Hardware checks stay CRITICAL on healthy hosts |
| [Containerized daemons and resource limits](container-resource-limits.md) | A containerized service misbehaves under load |
| [NGINX Plus upgrade preflight](nginx-plus-upgrade-preflight.md) | Upgrading NGINX Plus across licensing or memory changes |
