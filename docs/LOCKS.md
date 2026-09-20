# luc.lock profiles

The lockfile is restricted TOML owned by `luce-pkg`; compilers continue to read
`luce.toml` and never fetch packages during import resolution.

- Schema 1 preserves the original origin/name/version/source-digest/compiler pins.
- Schema 2 additionally pins the Git commit and numeric toolchain version.
- Schema 3 is the canonical dependency-graph profile. It retains every v2 field,
  adds compiler package identity, current package root `.`, publisher-key SHA-256,
  and the complete signed requirement/exact-selection edge set.

Schema-3 packages and their dependency edges use lexical coordinate order. The
decoder validates all coordinates and requirements, exact edge targets and
versions, source/key digest forms, compiler-identity uniqueness and acyclicity.
Unknown, duplicate or incomplete fields, and out-of-order package/dependency
tables, fail. The input is
bounded to1MiB,256 packages and128 direct dependencies per package. Encoding
validates the complete model into temporary storage before copying output, so an
error does not leave a partially updated destination.

Decoded strings and dependency slices are owned by `Lock`; callers close it once.
`Resolution` and lock generation keep separate responsibilities: resolution
chooses a deterministic graph from already authenticated LRS2 metadata, while the
lock records those choices. A lock does not authenticate its own publisher-key
fingerprints or establish registry freshness/rollback policy. Clients must verify
the actual key, signed release metadata and source bytes against independently
trusted policy before promoting a cache entry.
