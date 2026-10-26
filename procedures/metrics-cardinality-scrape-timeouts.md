# Metrics scrape timeouts caused by label cardinality

| At a glance | |
|---|---|
| **When to use** | Scrapes fail or time out while pods look healthy |
| **Impact if mishandled** | False outage alerts; restart masks an unfixed code defect |
| **Risk level** | Medium (production restart) |
| **Approval required** | Incident owner and service owner, scoped to one deployment-managed restart |
| **Escalate to** | Service code owner; image/release owner |
| **Last reviewed** | 2026-10 |

A failed `up{app="<service>"}` scrape is not proof the service is down. In one real incident a
forwarder's Prometheus metrics retained raw unmatched/404 request paths in histogram labels. Those
labels accounted for over 99.9% of samples, and scrapes alternated with ten-second timeouts at
236,000–379,000 samples and 26–42 MB responses, while both pods stayed Ready with zero restarts.

## Detection and scope

1. Verify exact pod readiness, restarts, service endpoints, and actual downstream response statuses.
2. Query each scraping agent's target list (for example, a VictoriaMetrics `vmagent` at
   `/api/v1/targets` through the authorized Kubernetes pod proxy). Filter to the service's label;
   retain `lastError`, scrape timestamp, duration, and sample count.
3. Inspect the live metrics endpoint for unbounded labels (raw paths, IDs, query strings).

## Authorized remediation and verification

1. Get explicit production approval from the incident owner and service owner, scoped to a
   deployment-managed restart. Approval for one incident is not standing approval for future incidents.
2. Capture the deployment strategy, replica count, probes, image tag and actual image digests, readiness,
   and service endpoints. Kubernetes readiness is not a downstream forwarding test; do not promise
   uninterrupted requests or two serving replicas throughout.
3. Using an authorized context, execute once:

   ```bash
   kubectl --context <ctx> -n <ns> rollout restart deployment/<name>
   kubectl --context <ctx> -n <ns> rollout status deployment/<name> --timeout=600s
   ```

   If the result is uncertain, inspect the restart annotation and generation before considering a
   retry. Do not manually delete pods, scale to zero, or change strategy as a substitute.
4. Verify both replacements are Ready, original pods are gone, service endpoints reference the
   replacements, and actual image digests are recorded. Stop and investigate read-only if scheduling,
   probes, or rollout fail; a rollback or strategy change needs its own authorization.
5. Verify the new targets on every collector with fresh timestamps, no scrape errors, and substantially
   reduced samples. Sustain observations across the alert's lookback window and evaluation cadence. In the
   real case, scrapes fell to 2–12 ms and 37–77 samples.
6. Check legitimate forwarding logs for actual downstream success, not merely application HTTP 200. Read
   back the incident state rather than equating rollout completion with mitigation.

## Exclusions and follow-up

- A restart clears process-local metric state; it is **not** a permanent cardinality fix. Ask the code
  owner to bound unmatched path labels while preserving existing dashboard dependencies.
- Downstream rejections that persist after restart are a separate problem; do not claim complete
  forwarding health or a customer outage from scrape results alone.
- An unchanged image tag does not guarantee identical software: replacement pods can resolve a different
  digest. Ask the image owner to investigate mutable-tag drift and evaluate digest pinning through the
  normal change process; do not silently roll back.
- Do not apply this procedure to unproven pod failures, unhealthy dependencies, or unrelated restarts.
