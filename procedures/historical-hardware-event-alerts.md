# Historical hardware-event alerts: archive, canary, and recover the real monitoring state

| At a glance | |
|---|---|
| **When to use** | Hardware checks stay CRITICAL on hardware that is currently healthy |
| **Impact if mishandled** | Erasing evidence of a real fault; repeated incident churn |
| **Risk level** | High (irreversible log deletion) |
| **Approval required** | Explicit approval per host for the clear; no carry-forward from prior cases |
| **Escalate to** | Operations on-call; plugin and runbook owners |
| **Last reviewed** | 2026-10 |

Some hardware checks (for example, a Nagios plugin reading the IPMI System Event Log) latch on old
asserted events and keep alerting CRITICAL on healthy hardware. In one real case, six hosts' incidents
reopened repeatedly after incident-only mitigation. Clearing the archived historical SEL events
recovered the actual checks, and all six incidents auto-resolved and stayed healthy through the next
natural check cycle.

## Detection criteria

- The alert quotes an old asserted event (for example, a PSU event); successive notifications retain the
  same problem/event IDs and CRITICAL state.
- Independently confirm present health: power inputs redundant, no chassis power/control/drive/cooling
  faults, application readiness, and service state/restart count. Old dates or a missing deassertion do
  not establish present health.
- Inspect the **deployed** check and helper, not just an assumed source repository. A plugin that
  rewrites the full SEL snapshot, deletes its seek file each run, and rescans historical assertions
  without pairing recovery events will alert forever.

## Where to investigate and whom to involve

| Field | Concrete action |
|---|---|
| Immediate next action | Verify live hardware and archive the exact host's full SEL before considering any reset |
| Systems/sources | Host IPMI and application status; inventory; the actual monitoring receiver's live status and service config; root crontab; deployed plugin and publisher |
| Search terms | Exact FQDN/service tag, check name, plugin filename, submission wrapper name |
| Operational owner | On-call operations team; existing follow-up ticket |
| Procedure/source owners | Runbook repository maintainer and plugin repository owner per repository metadata |
| Authorization | Explicit approval for clearing SEL on the exact healthy hosts; a prior investigation or incident-only mitigation is not approval to erase event history |
| Evidence to retain | Full pre-clear raw SEL, host/tag/time/SHA-256, live sensors/readiness, receiver state/timestamps, incident recovery, approval, any failed step |
| Stop condition | Current fault, changed or unarchived SEL, identity mismatch, failed backup hash, unverified runbook action mapping, unexpected post-clear record, or failure of the canary's upstream recovery |

## Remediation sequence

1. Confirm exact FQDN and physical service tag. Archive `ipmitool sel info`, `sel list`, `sel elist`, and
   `sel writeraw`, plus sensor/chassis health, readiness, timestamps, and plugin hash. Verify the archive
   SHA-256 after download. Do not treat a truncated chat excerpt as a full backup.
2. Obtain explicit authorization for the irreversible SEL deletion. If a UI-driven runbook exists, verify
   the deployed revision's action mapping before invoking it — a nominal "check" action that actually
   clears is a known class of bug.
3. Choose one host as a canary. Immediately before clearing, require the current SEL to match the reviewed
   archive and recheck live hardware/application health. Only then run `ipmitool sel clear`, followed by
   post-clear inspection and the actual check.
4. Expect the controller to need time after erase. A transient "unknown" response followed by one benign
   "log area reset/cleared" record is normal; use bounded, logged read retries and do not issue a second
   clear. An unexpected new event is a stop condition.
5. Require local plugin exit 0 / OK and continued power redundancy and readiness. A benign reset record
   means success does not require a literally empty SEL.
6. Trace the normal result-submission path. The check may run from root's crontab through a wrapper that
   `/etc/cron.d` and systemd timers do not reveal; inspect root crontab before concluding the collection
   job is absent.
7. Run the real check through that path to refresh monitoring, supplying the host's verified FQDN. Never
   fabricate an OK payload or inject a result directly into the monitoring system.
8. Verify the actual receiver accepts a fresh **HARD OK** after the clear, with notifications enabled and
   no acknowledgement/downtime. Only then proceed to the remaining approved hosts, preserving each host's
   independent gates and evidence.
9. Allow normal recovery notifications to resolve the incident; verify state by readback. If automation
   does not close a recovered incident, mitigate only with the demonstrated upstream recovery. Never
   repeatedly mitigate while the upstream check remains CRITICAL.
10. Verify the next naturally scheduled check also stays HARD OK, not just the manual refresh.

## Monitoring-vantage and cadence corrections

- Resolve the real backend through inventory and confirm `hostname -f`; a friendly monitoring hostname may
  point at a frontend with no matching services.
- A passive service whose fallback command returns a dummy stale-result warning will not execute the real
  check if forced. Use the configured collection/publisher path instead.
- Distinguish collection cadence from notification interval. Hourly re-notification does not prove a
  configured 60-minute notification interval.
- A local manual check returning OK does not update the receiver by itself. Verify the receiver's
  last-check time, output, and state, then verify the next scheduled submission.

## Exclusions

- Do not use this procedure to dismiss live PSU, disk/RAID, voltage, cooling, or other hardware faults,
  unknown/inaccessible sensors, unstable applications, or mismatched assets.
- Do not clear SEL without complete archived evidence and explicit authorization; no approval carries
  forward from a previous successful case.
- Do not remove seek files, globally suppress notifications, clear unrelated logs, change controller
  state, reboot, or treat event age as proof of recovery.
- Archiving and resetting a historical-event latch is an operational remediation, not a permanent plugin
  fix. File the plugin defect with its owner.

## Related: historical kernel disk-alert reset

The same pattern appears when a disk plugin reads the kernel ring buffer and alerts on months-old
media/predictive errors while the controller reports all volumes Optimal and all disks Online. The
temporary, explicitly authorized operational reset:

1. Establish that the plugin fails only on retained historical kernel records while current controller
   checks are clean (no rebuild, zero reported errors).
2. Obtain explicit approval for the exact host and a one-time clear of the **entire** in-memory kernel
   buffer. Do not delete `/var/log/kern*`, clear SEL, reboot, or modify controller state as a substitute.
3. Archive the complete raw kernel buffer durably before clearing; retain its byte count and SHA-256.
4. Immediately before the mutation, verify host identity, fresh controller health, and that the buffer
   still matches the archive. Run `dmesg --clear` **once**. If the result is uncertain or a new error
   appears, inspect; do not repeat.
5. Run the actual local plugin and require exit 0; a local OK alone does not update monitoring.
6. Request one immediate scheduled execution through the normal monitoring command path; never inject a
   fabricated OK.
7. Read back a fresh HARD OK newer than the clear, with notifications enabled.
8. Recheck at the next natural monitoring cycle. Do not claim scheduled-cycle validation prematurely.
