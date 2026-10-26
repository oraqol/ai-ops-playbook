# NGINX Plus upgrade preflight (R33+ licensing, R37 upstream memory)

| At a glance | |
|---|---|
| **When to use** | Upgrading NGINX Plus across licensing or shared-memory changes |
| **Impact if mishandled** | Failed package install or upstream memory exhaustion at reload |
| **Risk level** | High (edge traffic) |
| **Approval required** | Change approval for the upgrade and canary plan |
| **Escalate to** | Load-balancer platform owner |
| **Last reviewed** | 2026-10 |

Before moving an NGINX Plus R32 host to an R33+ configuration-management branch:

1. Confirm `/etc/nginx/license.jwt` exists, is non-empty, owned by `root:root`, and mode `0600`.
2. Ensure configuration management places the (secret-store-backed) license file **before** installing
   the package; otherwise the R33+ package pre-install check aborts and can leave new modules unpacked
   beside an old core.
3. Inspect **every** active upstream-generation path, not only the legacy generator (for example, both a
   script-based generator and a templated `upstream.conf.j2`).
4. Require all generated upstream zones to use the size proven safe for the target release before reload.
   In R37, `2048k` shared-memory zones were exhausted (`ngx_slab_alloc() failed: no memory`) on hosts
   with large upstreams; `4096k` was required:

   ```bash
   grep -H 'zone .*_zone' /etc/nginx/upstream/*.conf
   sudo nginx -t
   ```

5. Verify `dpkg --audit` is empty, NGINX and NGINX Agent are active, the new master is running, and live
   TLS still serves the expected certificates.
6. Stop if package versions are mixed, `nginx -t` fails, or a generator rewrites `4096k` zones back to
   `2048k`.
