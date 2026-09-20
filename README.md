# luce-pkg

Native Luce Base package identity, dependency resolution and `luc.lock` handling.
MIT OR Apache-2.0. Compilers still read `luce.toml`; this package owns the
lockfile. No foreign solver.

Experimental. Not a complete installer, TUF client or registry.

`pkg.resolve_graph` is the bounded offline LRS2 graph solver. It accepts root
exact/caret requirements plus a same-origin universe of canonical LRS2 releases,
then selects one release per registry coordinate. The result is independent of
input order: unresolved coordinates are considered lexically and versions are
tried highest-first. The solver accumulates every transitive constraint and
backtracks when a high version makes the remaining graph impossible. It rejects
unsatisfied intersections, dependency cycles, duplicate coordinate/version
candidates and two registry coordinates that claim the same compiler package
identity. Results are returned in lexical coordinate order as indices into the
caller's catalog. Bounds are128 roots,256 selected packages,4,096 candidates and
67,108,864 work units; exceeding any bound fails rather than returning a partial
graph. Catalog and decoded-metadata storage must remain alive until the owning
`Resolution` is closed. This API performs no network access and does not establish
publisher trust; callers supply already authenticated metadata.

`pkg.encode_release_v2` defines signed LRS2 metadata. It retains the LRS1 origin,
registry coordinate, version, Git commit, source SHA-256, language and toolchain
fields, and additionally binds the compiler package identity plus a complete
dependency declaration. Each dependency is a same-origin registry coordinate and
an exact or caret numeric-semver requirement. Entries must be unique and sorted by
coordinate, so equivalent graphs have one byte representation. A release cannot
depend directly on itself. Bounds are128 dependencies,97-byte coordinates,
63-byte requirements and24,576 bytes of total metadata. `Release.dependency_at`
returns borrowed validated entries. ML-DSA verification covers these bytes and
the source digest exactly.

Legacy LRS1 remains decodable and verifiable for existing releases, but it has no
compiler package identity or dependency declaration and therefore cannot be used
as evidence of a complete dependency graph. A programmatic zero schema continues
to encode as LRS1 for source compatibility; decoded releases explicitly report
schema1 or2. Both schemas reject noncanonical framing and trailing data before
use. LRS2 declares dependencies but does not by itself resolve them, establish
publisher trust/freshness, or prove that a source manifest matches the declaration;
`resolve_graph` and the publication manifest validator are separate explicit gates.

`pkg.validate_release_manifest` closes the shared source-agreement half of that
publication gate. For LRS2, the root `luce.toml` must contain matching `[package]`
`name` and `language` values plus the complete dependency set under
`[registry.dependencies]`, for example `"acme/core" = "^1.2.0"`. Quoted
coordinates keep the file valid TOML; existing compilers ignore this new table.
Declaration order is irrelevant, but duplicates, malformed coordinates or
requirements, missing/extra entries, identity mismatches and legacy LRS1 are
rejected. The parser is bounded to1MiB and128 dependencies and does not mutate
the manifest. Server-side Git-tree extraction remains the caller's responsibility.

`pkg.decode_manifest` exposes the same restricted compiler-compatible declaration
to package clients. It returns package identity, language and a lexical owning
array of `[registry.dependencies]` entries, so resolution and publication share
one parser. Returned strings borrow the caller's manifest bytes; call
`Manifest.close` to release its dependency array.

`pkg.encode_versions` / `pkg.decode_versions` define the canonical LPV1 catalog
used for registry version discovery: `LPV1`, a u16le count (maximum1024), then
u8-length canonical numeric semantic versions in strict descending order. Exact
framing, uniqueness, order, numeric bounds and trailing bytes are checked before
selection. `pkg.select_version` accepts an exact version or a leading-`^` caret
requirement and deterministically returns the highest compatible catalog text.
The returned catalog views borrow their input. A catalog is advisory and does not
replace verification of signed release metadata/source, publisher trust, catalog
freshness or rollback protection.

Numeric version components range from0 through18446744073709551615 (`u64`).
Overflow is rejected before arithmetic, including in lockfiles, release metadata
and dependency candidates; hostile oversized components return validation errors
rather than triggering checked-arithmetic traps. Prerelease/build suffixes remain
unsupported by this numeric version profile.

The current encoder rejects quotes, backslashes and control bytes in literal
fields to prevent TOML injection; invalid input leaves the output buffer unchanged.
It uses restricted v1/v2 TOML profiles, not general TOML syntax. `decode_lock`
validates the full document and returns a closeable owner of all decoded strings;
`lock_origin` validates the same document but returns a string borrowing its input.
Headers must precede packages; schema 1 or 2 and one origin are required. Each package
requires name, numeric version, digest and compiler exactly once. Duplicate package
names, unknown keys/tables, escapes and incomplete entries are rejected. Blank
lines, full-line comments, horizontal whitespace and CRLF are supported. Limits
are 1 MiB of input and 4096 packages. Digest/identity policy, release signature
verification and remote installation are separate requirements, not established
by successful parsing.

Schema 2 additionally requires a lowercase 40-hex Git `commit` and numeric semantic
`toolchain` version on every package. `Lock.schema_version` exposes the decoded
version. `encode_lock(..., schema_version=2)` emits these pins; the default remains
v1 for compatibility. Encoding pins as v1 fails instead of silently dropping them.
V1 reads initialize both pin fields empty. Neither format establishes publisher
trust or proves that a toolchain version uniquely identifies compiler binaries.

Schema 3 is the canonical complete-graph lock profile. Packages must be in lexical
coordinate order and add compiler `package_name`, current root `.` and the lowercase
SHA-256 fingerprint of the independently trusted publisher key. Each nested
`[[package.dependency]]` records the signed exact/caret requirement and the exact
selected version. Dependencies are lexical and unique; every edge must target a
locked package at that exact version and satisfy its requirement. The decoder also
rejects missing targets, cycles, duplicate compiler identities and malformed
coordinates/digests before exposing the lock. Limits are1MiB,256 packages and128
edges per package. V1/v2 reject v3 fields rather than silently discarding them.
Successful v3 parsing verifies internal graph consistency, not signatures,
publisher authorization, catalog freshness or rollback protection.

```sh
python3 tools/bootstrap.py
python3 tests/run.py
```
