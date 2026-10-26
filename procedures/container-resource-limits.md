# Containerized daemons and resource limits

| At a glance | |
|---|---|
| **When to use** | A containerized service degrades or drops work under load |
| **Impact if mishandled** | Silent request loss; hotfix reverted by config management |
| **Risk level** | Medium (service restart) |
| **Approval required** | Service owner for restart; config-management change for the permanent fix |
| **Escalate to** | Service owner; config-management module owner |
| **Last reviewed** | 2026-10 |

When a containerized service misbehaves under load, check inherited limits before tuning the
application.

1. **Compare the container's limit to what its config requests.**
   `docker inspect <name> --format '{{json .HostConfig.Ulimits}}'` against application settings such as
   cache sizes and maximum sessions. An empty `Ulimits` array means Docker's default `NOFILE=1024`,
   regardless of the systemd unit's `LimitNOFILE`.
2. **Confirm saturation directly:** `ls /proc/<pid>/fd | wc -l` versus `Max open files` in
   `/proc/<pid>/limits`. Equal values are exhaustion, not coincidence.
3. **After the fix, verify the count settles _above_ the old ceiling.** That is the proof the limit was
   binding and that work was previously being dropped.
4. **Check whether the file you patched is configuration-managed** (for example, a
   `# This file is managed by Puppet` header). If so, the hotfix is temporary; the permanent change must
   land in the module or the next agent run reverts it. File that as the primary action item, not a
   footnote.
5. **Apply the same check to peer hosts** built from the same template before they cross the threshold.
