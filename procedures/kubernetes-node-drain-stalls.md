# Runbook-automation execution and stalled Kubernetes drains

## Before requesting credentials for a runbook-automation API

1. From the intended execution host, resolve the API endpoint and make a read-only probe. A
   `404 Cannot GET` on an unsupported route still proves the application is reachable; a DNS failure or a
   gateway response does not. Do not assume an older hostname still serves the API.
2. Retrieve the authoritative runbook source and input schema. Confirm the exact branch, target hostname
   format, flags, stop conditions, and expected child runbooks.
3. Confirm the change is approved and record the approver and approval timestamp.
4. Collect the service password only in a secure interactive prompt. Never place it or the returned token
   in chat, command arguments, shell history, files, logs, or ticket comments.
5. When the workstation cannot reach the API, stream a minimal credential payload over encrypted SSH stdin
   to an internal support host, run the API client there, and return only the job ID. Do not copy a
   credential file or token to the support host.

## Monitoring a cordon-and-drain runbook

Monitor the parent task and every child task. Require exit code `0`; do not treat submission or a
successful cordon as completion.

If the task remains at "draining node" longer than the normal eviction interval:

1. Inspect the active `kubectl drain` process and query the target node's remaining pods and
   PodDisruptionBudgets from the same worker network and mount namespace the automation uses.
2. Separate DaemonSets and static mirror pods from application workloads.
3. For each remaining application pod, record namespace, owner, phase, deletion timestamp, container
   state, IP, exit code, finalizers, and events.
4. Stop and escalate on a live unmanaged workload, PDB denial, replacement-capacity failure, cluster
   degradation, or an unfamiliar blocker.
5. A force deletion is allowed only when the approved change already includes forced drain **and**
   evidence proves the pod is dead and stale: already terminating, no running container, no IP, completed
   successfully or otherwise nonfunctional, and no new events. Delete only that exact pod with
   `--grace-period=0 --force`; never broaden the deletion.
6. Re-read the original task. Require the drained confirmation, a successful deactivation child, and
   parent exit code `0`.

## Post-change verification

- Node is `SchedulingDisabled`.
- No non-DaemonSet application pods remain.
- Expected DaemonSets and static mirror pods are identified rather than misreported as blockers.
- Configuration management is disabled on the node.
- kubelet is stopped and disabled at boot.
- Monitoring is silenced/inactivated.
- The provisioning system reports the node inactive.
- Inventory reports it reserved and unmonitored.
- The change, repair ticket, and incident contain the automation job IDs and validation evidence.

Do not use a single inventory `active` column as the sole deactivation test; reserved, unmonitored
servers can retain `active=1`. Use the provisioning activation state plus inventory reserved/unmonitored
status as the service-removal evidence.
