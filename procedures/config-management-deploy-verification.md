# Verifying configuration-management deployments (including AI-agent branches)

| At a glance | |
|---|---|
| **When to use** | A change, including an AI-agent branch, "should be live" but is not |
| **Impact if mishandled** | Believing a fix is deployed when the fleet still runs old config |
| **Risk level** | Low (read-only verification) |
| **Approval required** | None to verify; normal change approval to push |
| **Escalate to** | Config-management/pipeline owner |
| **Last reviewed** | 2026-10 |

A green pipeline is not proof that a change reached the fleet. This matters most for branches opened by
AI coding agents, which may use naming conventions the deployment tooling silently ignores.

For "branch not visible" or "stale production" reports in a Puppet/r10k control repository:

1. Before pushing, validate the branch name against the patterns the environment generator accepts (for
   example `copilot/swe-[a-z0-9-]+` or `[a-z0-9_]+`). Do not assume a generic hyphenated branch is
   deployable.
2. Verify the source ref and commit in the control repository.
3. Verify the normalized environment branch in the r10k mirror; r10k may strip `/` and `-` from branch
   names.
4. Inspect the deploy pipeline run and retain the automation job ID.
5. Query every child job and require exit code `0`. Do not rely only on the pipeline result if its
   notification step logs child failures without failing.
6. Validate the affected datacenter's live file under
   `/etc/puppetlabs/code/environments/<environment>/...`.
7. Compare its SHA-256 with the exact source ref. Validate `production` separately if branch CI excludes
   production pushes.

## Completion

Matching source and deployed hashes on a Puppet primary in an affected datacenter — not merely an r10k
branch or a green pipeline.
