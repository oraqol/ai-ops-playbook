# Empty results are a vantage-point claim, not a fact

A "not found", "unreachable", or zero-row result usually describes where you queried from, not what
exists. This applies equally to commands you run and to answers an AI agent gives you. Before
concluding absence:

1. **Re-run from the context where the evidence was actually seen.** If someone pastes output with a
   prompt or hostname in it, that host is the authoritative vantage point. Regionally scoped inventories
   (for example, a GSLB domain list) can return rows from one region's support host and zero from
   another's, so the wrong host yields a confident false negative.
2. **Enumerate candidate access paths before declaring a host down.** A host can be healthy while direct
   SSH and one bastion both time out, yet reachable through a different jump host. Failure from one or
   two paths is not evidence of an outage.
3. **Vary the query form.** Fully quoted exact-phrase searches can return irrelevant results where plain
   unquoted keywords return the hit. Retry with a different phrasing before reporting nothing found.
4. **Say which vantage point produced the result.** Report "no matches from `<host>`" rather than
   "does not exist", so the gap is visible and correctable.

## Network-segmented estates

When a group of hosts is not reachable the same way as the rest of the fleet, resolve access in order
rather than concluding a host is unreachable:

1. Check whether the source can ICMP-ping the host but has port 22 filtered — that makes a healthy host
   look down.
2. Check whether the VPN profile actually carries the target subnet before blaming the host:

   ```powershell
   Get-NetRoute -AddressFamily IPv4 | Where-Object { $_.DestinationPrefix -like '10.*' }
   ```

3. Use a jump host that can reach the segment. Pass remote scripts base64-encoded so quoting survives
   both hops:

   ```powershell
   $b64 = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($inner))
   ssh -A -o BatchMode=yes <user>@<jump-host> `
     "echo $b64 | base64 -d | ssh -o BatchMode=yes <user>@<target-ip> bash -s"
   ```

4. In WSL, `SSH_AUTH_SOCK` is unset in new shells. Find it with `ls -1 /tmp/ssh-*/agent.*` and confirm
   keys with `ssh-add -l` before assuming an authentication failure.

Raise any missing VPN route with the network owners so the segment is directly reachable during on-call.
