# Workflow Improvement Playbook

Use this document to identify recurring execution problems and apply a concrete correction. Update the
[encounter log](templates/encounter-log.md) whenever a new process exposes one of these patterns.

The playbook is written for infrastructure and operations work done by people, by AI agents, or by a
person working through an agent. Every rule applies equally to all three.

## Contents

- [Recommendation standard](#recommendation-standard)
- [1. Reactive planning](#1-reactive-planning)
- [2. Excessive manual supervision](#2-excessive-manual-supervision)
- [3. Reliance on conversational memory](#3-reliance-on-conversational-memory)
- [4. Weak ownership discovery](#4-weak-ownership-discovery)
- [5. Premature conclusions](#5-premature-conclusions)
- [6. Incremental decision overhead](#6-incremental-decision-overhead)
- [7. Late documentation and redaction](#7-late-documentation-and-redaction)
- [8. Command-handling errors](#8-command-handling-errors)
- [9. Poor long-running-work boundaries](#9-poor-long-running-work-boundaries)
- [10. Failure to convert discoveries into reusable methods](#10-failure-to-convert-discoveries-into-reusable-methods)
- [Reminder behavior](#reminder-behavior)
- [Concrete procedures](procedures/)

## Recommendation standard

Do not accept advice such as "find the owner," "check documentation," or "get approval" without an
executable next step. Every recommendation must specify:

1. **Where to search:** work-tracking items, repositories and wikis, chat, email, service inventory,
   monitoring, the provisioning system, or hardware management.
2. **What to search for:** exact hostname, device ID, service name, error or substate, cluster ID, ticket
   number, command, or configuration key.
3. **Whom to contact:** a named person when evidence identifies one; otherwise the specific owner role,
   queue, or team.
4. **What to ask:** the exact decision, evidence, access, or action required.
5. **What authorization is required:** ticket comment, change approval, service-owner signoff, network
   approval, or platform-owner approval.
6. **What proves completion:** an explicit command result, state transition, owner response, ticket
   comment, or validation test.

### Required recommendation format

| Field | Required content |
|---|---|
| Immediate next action | One action that can be performed now |
| System/source | Exact tool or data source to use |
| Search/query | Exact identifiers and search terms |
| Person/team | Named owner or precise owner role/team |
| Request | Exact question or authorization needed |
| Evidence to retain | Link, comment, output, timestamp, or screenshot |
| Stop condition | Condition that prevents further execution |

## 1. Reactive planning

### Warning signs

- Asking "what next?" repeatedly during execution.
- Discovering required access, dependencies, or approvals after work starts.
- Changing the plan after each command rather than working from a complete sequence.

### Required method

Create an [execution charter](templates/execution-charter.md) before making changes:

1. Objective and scope.
2. Systems and hosts affected.
3. Authoritative ticket or approval.
4. Owners and escalation paths.
5. Preconditions and access requirements.
6. Ordered execution steps.
7. Validation after each major stage.
8. Completion criteria.
9. Rollback procedure.
10. Stop conditions and handoff point.

## 2. Excessive manual supervision

### Warning signs

- Repeated countdown or status requests.
- Checks occurring more frequently than the expected stage duration.
- Monitoring healthy automation without evidence that intervention is needed.

### Required method

Define monitoring from measured behavior:

- Record normal duration and maximum expected duration.
- Check at meaningful stage boundaries or at 75–100% of normal duration.
- Use event-driven notifications when available.
- Define an "intervene only if" threshold.
- Report ongoing progress without changing the plan.

## 3. Reliance on conversational memory

### Warning signs

- Important commands, fixes, or decisions exist only in chat.
- A future session would need to reconstruct the current state.
- The same discovery is researched more than once.

### Required method

Maintain a durable state record containing:

- Current phase and active target.
- Completed targets and timestamps.
- Exact validated commands.
- Expected and actual results.
- Known failure signatures and approved remediations.
- Remaining work and the next safe action.

Put operationally authoritative information in the approved ticket or runbook, not in a chat transcript.

## 4. Weak ownership discovery

### Warning signs

- Searching for owners only after a blocker occurs.
- Uncertainty about which team approves, executes, or validates a step.
- Work pauses while escalation channels are located.

### Required method

Query available enterprise sources before execution:

1. Search work-tracking items for the exact hostname, device ID, service name, cluster ID, and failure
   substate. Inspect assignees, commenters, approvers, area path, linked changes, and successful
   historical tickets.
2. Search chat for the same identifiers plus phrases such as `owner`, `on-call`, `decommission`,
   `reinstall`, `rollback`, and the exact error.
3. Search email for prior approvals, incident threads, maintenance announcements, and messages from
   people found above.
4. Search the relevant repositories and wiki for `CODEOWNERS`, service metadata, runbooks, inventory
   declarations, configuration-management roles, deployment pipelines, and escalation aliases.
5. Compare the evidence. Do not treat a single old message as current ownership.

Build a RACI-style [ownership map](templates/ownership-map.md) from the results:

| Function | Concrete request |
|---|---|
| Service owner | "Confirm these exact hosts/services have no remaining consumers and approve the stated impact." |
| Platform owner | "Confirm this runbook and profile are correct for these device IDs, including known failure recovery." |
| Network owner | "Confirm BGP/LACP/VIP removal is complete and state who may remediate failed network automation." |
| Monitoring owner | "Approve monitoring suppression and state the required closure or retirement status." |
| Security/identity owner | "Confirm certificate, secret, Kerberos, or access consequences and approve the handling method." |
| Change approver | "Approve the exact destructive boundary, rollback limit, execution window, and stop conditions." |
| Escalation channel | Post hostname/device ID, ticket, error, timestamps, completed diagnostics, and the exact action requested. |

Record each owner with a supporting link or inventory source. If no owner can be proven, create or
update a task assigned to the likely owning queue and explicitly request ownership confirmation before
destructive work.

### Ownership-question procedure

Whenever ownership of any task, component, host, service, configuration, approval, or remediation is
requested:

1. Search work items using exact hostnames, device IDs, service names, error text, repository names, and
   related ticket IDs. Extract assignees, approvers, commenters, area paths, linked teams, and prior
   operators.
2. Search chat for the same identifiers and for `owner`, `on-call`, `SME`, `escalate`, `support`, and
   `approver`. Identify who answered similar questions and which channel handled them.
3. Search email for maintenance notices, approval threads, incident escalations, distribution lists, and
   prior conversations involving the people or teams found above.
4. Search repositories, wikis, service metadata, `CODEOWNERS`, monitoring configuration, deployment
   pipelines, and inventory for formal ownership references.
5. Search available directory information to determine each person's title, team, manager chain, and
   organizational relationship when that data is accessible.
6. Cross-check dates and sources. Label ownership as **confirmed**, **historically indicated**, or
   **inferred**.

The response must include:

| Section | Required detail |
|---|---|
| Primary owner | Named person and team, or the exact owning queue if no person is confirmed |
| Why they own it | Work-item, chat, email, repository, inventory, or directory evidence |
| Organizational placement | Team purpose, parent organization, adjacent teams, and manager chain when evidenced |
| Responsibility boundary | What this owner controls and what belongs to another team |
| First contact | Person, alias, queue, or chat channel to contact first |
| Exact request | A ready-to-send question containing identifiers, evidence, impact, and desired action |
| Authorization | Who must approve and where approval must be recorded |
| Escalation order | Primary owner, team/on-call, manager or service lead, dependent team, and incident/change escalation |
| Uncertainty | Missing or conflicting evidence and the next query needed to resolve it |

Do not invent an org chart from job titles or message participation. If the available evidence does not
establish reporting relationships, state that explicitly and provide the exact directory, email, chat, or
work-item query needed next.

### Non-blocking workflow coaching

Workflow feedback must not interrupt active work with quizzes, scores, artificial deadlines, or requests
for the operator to demonstrate knowledge.

When a less efficient method appears:

1. Continue the operational task unless safety or authorization genuinely requires a decision.
2. Briefly identify the inefficient method.
3. State the concrete better method, including exact systems, queries, owner/team, authorization, and
   evidence where relevant.
4. Apply the better method directly when authorized and safe.
5. Add durable, reusable improvements to this playbook.

Feedback should be concise during execution and more detailed at natural stopping points. Ask the
operator only for genuine requirements, authorization, credentials, or high-impact design decisions —
not as a teaching exercise.

## 5. Premature conclusions

### Warning signs

- Treating one symptom as proof of root cause.
- Attributing an outage to the current change without a baseline.
- Declaring success from a control-plane state without checking the resulting system.

### Required method

Use an evidence ladder:

1. Record the pre-change baseline.
2. Capture the observed symptom and timestamp.
3. Verify the control-plane state.
4. Verify the target system state.
5. Check dependencies and external services.
6. Compare against known-good or historical behavior.
7. State what is proven, likely, and still unknown separately.

## 6. Incremental decision overhead

### Warning signs

- Requesting approval for every routine verification.
- Handling closely related questions one at a time.
- Repeatedly revisiting decisions that could have been defined in advance.

### Required method

Prepare a decision package:

- Proposed sequence.
- Commands grouped by reversible stage.
- Verification after each group.
- Explicit destructive boundary.
- Known remediation authority.
- Conditions that require another approval.

Batch routine read-only checks and preserve separate approval for destructive actions.

## 7. Late documentation and redaction

### Warning signs

- Reconstructing commands and results after the project.
- Discovering sensitive content after posting.
- Producing closure documentation from memory.

### Required method

Keep a live sanitized execution log:

- Timestamp each action.
- Record commands without credentials or secret values.
- Capture concise results and validation evidence.
- Redact keys, tokens, cookies, passwords, and unnecessary personal identifiers before posting.
- Update the authoritative ticket at major transitions.

## 8. Command-handling errors

### Warning signs

- Multiple commands accidentally concatenated.
- Variables or quoting behave differently from the displayed example.
- A failed command is followed by another command before its effect is understood.

### Required method

- Provide one executable command block per logical action.
- State the expected output or exit condition.
- Review destructive parameters before execution.
- Run, inspect, and validate before proceeding.
- Prefer scripts or established runbooks for repeated multi-host work.

## 9. Poor long-running-work boundaries

### Warning signs

- Work continues into an unattended period without assigned ownership.
- Multiple hosts remain in intermediate states.
- The next operator cannot determine the next safe action.

### Required method

Before a handoff or stopping point:

1. Finish the currently active unit when safe.
2. Return it to a defined stable state.
3. Do not start another unit.
4. Record current status and remaining targets.
5. Record the next command and its prerequisites.
6. Disable unnecessary polling or automation.

## 10. Failure to convert discoveries into reusable methods

### Warning signs

- A successful remediation remains an informal chat instruction.
- The same failure requires fresh diagnosis on the next host.
- Detection criteria are not separated from remediation steps.

### Required method

For every proven fix, document (see the [procedure template](templates/procedure.md)):

- Failure signature.
- Evidence required to confirm the cause.
- Conditions where the fix is safe.
- Exact remediation.
- Verification that proves recovery.
- Conditions where the fix must not be used.

## Reminder behavior

When one of these warning signs appears, explicitly identify:

1. The bad habit currently occurring.
2. Why it creates risk or wasted effort.
3. The exact source to query and exact search terms.
4. The named person/team or precise owner role to contact.
5. The exact question or authorization to request.
6. The evidence that must be retained.
7. The smallest immediate action needed to improve the workflow.
