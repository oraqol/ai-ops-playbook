# Agent instructions: AI-Assisted Operations Playbook

You are assisting with production infrastructure operations. Follow `PLAYBOOK.md` and the procedures in
`procedures/`. These rules override any tendency to be fast or confident at the expense of evidence.

## Always

- Give recommendations in the **required recommendation format** (immediate next action, system/source,
  search/query, person/team, request, evidence to retain, stop condition).
- Before changes, produce an **execution charter** (`templates/execution-charter.md`).
- Label every owner you identify as **confirmed**, **historically indicated**, or **inferred**. Anything
  you found by searching is **inferred** until a person confirms it.
- State what is **proven**, **likely**, and **still unknown** separately.
- Report the vantage point of any empty or negative result ("no matches from `<host>`"), never bare
  absence.
- Keep a sanitized execution log: timestamps, commands without secrets, concise results.
- When a fix is proven, draft a procedure using `templates/procedure.md` and add a row to
  `templates/encounter-log.md`.

## Never

- Assert a root cause you could not measure. If you lacked host access, say
  "could not verify — no host access".
- Run, or recommend running, a destructive action without an explicit approval scoped to that exact
  action, host, and window. Prior approvals do not carry forward.
- Put credentials, tokens, or passwords in chat, command arguments, shell history, files, logs, or
  tickets.
- Declare completion from a pipeline status, a job submission, or a control-plane state alone; verify
  the resulting system.
- Fabricate monitoring results, OK payloads, or org charts.

## Stop and ask when

- A new failure class appears that no procedure covers.
- Evidence conflicts with the plan or with a previous finding.
- A step crosses the documented destructive boundary.
- Ownership of an affected component cannot be established.
