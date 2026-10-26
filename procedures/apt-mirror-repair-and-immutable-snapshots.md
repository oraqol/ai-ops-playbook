# APT mirror repair and immutable snapshotting

Use this procedure when an internal APT mirror (for example, of NodeSource) contains a zero-byte or
checksum-invalid package and a new pinned snapshot is required.

## Detection criteria

1. Identify the exact repository, version, package filename, expected size, and SHA-256 from the
   upstream `Packages` metadata.
2. Confirm the internal package is missing, zero bytes, the wrong size, or checksum-invalid.
3. Check for an active mirror process before starting another run.
4. Run the repository's normal `apt-mirror` command once and retain its log. A refresh is **not**
   successful if it reports `Unable to download`, a nonzero error count, `Metadata movement skipped`, or
   `Some files were not downloaded`.

## Remediation

1. If the upstream repeatedly resets the affected HTTP/2 streams, download only the failed packages with
   `curl --http1.1 -fLR --retry ...`.
2. Verify each temporary file against the size and SHA-256 from upstream metadata before installing it
   into the internal mirror.
3. Preserve the upstream `Last-Modified` timestamp when copying the file into place. Without the original
   timestamp, `apt-mirror` may treat the valid file as changed, truncate it, and retry the failing HTTP/2
   transfer.
4. Restore the repository-standard owner, group, and mode.
5. Rerun the normal `apt-mirror` command. Require every pool file to be reported as downloaded or
   unmodified, zero errors, and `Metadata moved`.
6. Confirm the published internal `Packages.gz` references the required package version, size, filename,
   and SHA-256.
7. Dry-run the snapshot against the exact repository-specific source and destination roots.
8. Create a dated immutable snapshot without moving any `latest` marker (for example, a `--no-mark`
   option), so no consumer pointer changes implicitly.

## Verification

- The live mirror and snapshot package have the expected size and SHA-256.
- Their inode numbers match and link count is at least two, proving the snapshot is a hard-linked
  immutable copy.
- The snapshot's `Packages.gz` contains the expected package stanza.
- The published `InRelease` URL returns HTTP 200.
- Downloading the package through the published snapshot URL produces the expected SHA-256.
- The ticket records the snapshot URL, package version, size, hash, refresh result, and whether any
  marker changed.

## Exclusions and stop conditions

- Never snapshot a repository while the required package is zero bytes, checksum-invalid, or absent from
  published metadata.
- Do not manually publish metadata after a failed mirror run; repair the content and require the normal
  mirror job to publish it.
- Do not start a duplicate mirror process.
- Do not use this HTTP/1.1 repair for unrelated download failures without first identifying the exact
  failed files and authoritative upstream checksums.
- Do not update `latest`, `live`, or another mutable marker unless the ticket explicitly authorizes
  redirecting consumers.
