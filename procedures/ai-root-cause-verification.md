# Verifying AI-generated root causes

AI SRE agents and automated triage bots can emit confident root causes even when they could not reach
the host. Treat an unverified cause as a hypothesis, not a finding.

## Procedure

1. **Check whether the agent actually had access.** Phrases like "SSH tools were not exposed",
   "504 Gateway Timeout", or "manual on-host validation was recommended" mean nothing was measured.
2. **Re-derive the cause from first principles before acting.** In one real case an agent asserted that
   journal growth was filling a disk. `journalctl --disk-usage` returned 8 MB while
   `/var/log/daemon.log` was 51 GB. Running the recommended `journalctl --vacuum-size` would have
   reclaimed nothing and produced a fifth repeat incident.
3. **For any "recurring alert that auto-mitigates", establish what actually cleared it.** Repeated short
   self-resolution usually means a scheduled job (logrotate, cron, cache eviction) is masking an unfixed
   cause, and shrinking intervals between recurrences mean it is getting worse.
4. **Measure the rate, not just the level.** `stat -c %s` on the growing file before and after a
   10-second sleep converts "disk is full" into a time-to-exhaustion number that justifies the severity.
5. **Feed the correction back to the agent's owners.** Ask them to make the tool report
   "could not verify — no host access" instead of asserting a cause, so on-call is not led toward a
   wrong remediation.

## Evidence to retain

- The agent's original claim and the access it reported.
- The independent measurement that confirmed or refuted it.
- The rate measurement and projected time to exhaustion.
- The feedback sent to the agent's owners.
