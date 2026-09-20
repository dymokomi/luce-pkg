# luce-pkg

Native Luce Base package identity, caret resolution and `luc.lock` handling.
MIT OR Apache-2.0. Compilers still read `luce.toml`; this package owns the
lockfile. No foreign solver.

Experimental. Not a complete installer, TUF client or registry.

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

```sh
python3 tools/bootstrap.py
python3 tests/run.py
```
