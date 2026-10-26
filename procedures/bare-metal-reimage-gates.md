# Bare-metal wipe and reimage gates

| At a glance | |
|---|---|
| **When to use** | Sequential wipe, reimage, or retirement of physical hosts |
| **Impact if mishandled** | Data loss or wrong host wiped; network left half-configured |
| **Risk level** | High (destructive, irreversible) |
| **Approval required** | Change record covering profile, wipe, and IP handling |
| **Escalate to** | Bare-metal/provisioning owner; network owner |
| **Last reviewed** | 2026-10 |

## Before each host

1. Query the provisioning system by device ID and record active/monitored flags, IP, kickstart profile,
   PXE state, LACP state, and hosted-instance count.
2. Query the work-tracking system using the device ID, hardware hostname, and any prior failed substate.
   Read at least one successful reinstall and any tickets matching the failure.
3. Search chat and email for the hardware hostname and failure substate to identify the bare-metal owner
   or operator who previously resolved it.
4. Confirm the change record authorizes the profile, destructive wipe, IP preservation method, and any
   pre-authorized remediation.
5. Use the BMC console (iDRAC/iLO) to verify chassis health, boot target, job queue, and console
   availability before assuming the tracker identifies the root cause.

## Completion evidence

- Provisioning history reaches `running`.
- PXE boot is disabled.
- LACP is restored.
- Expected IP and kickstart profile remain assigned.
- Monitoring remains disabled.
- The server is returned to the inactive state with a change-referenced retirement comment.

For automated polling, determine terminal state from the newest **state-history** row. Do not rely on
summary `state`/`substate` display columns; they can be blank after a server has reached `running`.

## Pre-authorized automatic retry

A forced reimage retry may be pre-authorized for a specific, well-understood failure signature (for
example, a first-boot or BMC-settings failure) when **all** of these are confirmed:

1. The server is in exactly that failure substate.
2. OS installation, post-install, and PXE disable completed first.
3. The expected IP and kickstart profile remain assigned.
4. Monitoring remains disabled.
5. Hardware/chassis checks show no active fault.
6. The BMC is reachable; collect the job queue and boot-state evidence when available.
7. No other reinstall is active in the sequential rollout.

Do not request permission again when this exact signature is confirmed. Record the failure evidence and
retry in the authoritative change/ticket.

Do **not** apply this recovery to:

- network-commit failures without proof that PXE is functioning;
- PXE/DHCP/TFTP failures;
- hardware, disk, power, or cooling faults;
- unexpected IP, VLAN, customer, or profile drift;
- an inaccessible BMC;
- failures before OS/post-install completion;
- any unrelated substate or new failure class.

After retry, require `running`, PXE disabled, restored LACP, expected IP/profile, monitoring disabled, and
final deactivation before advancing to the next host.

## Long-running rollout

- Process one host at a time.
- Use measured wipe duration to set check intervals.
- Automatically advance only after all completion criteria pass.
- Stop on a new failure class rather than improvising.
- Establish an explicit end-of-day boundary.
