# LRS1/LRS2 release binding

`pkg.encode_release` produces the message signed with ML-DSA-65. Byte layout:

1. ASCII `LRS1`, followed by little-endian u16 schema `1`.
2. Six text fields in order: origin, package coordinate, version, Git commit,
   source language, toolchain constraint. Each is u16 little-endian byte length
   followed by that many bytes. No terminators or padding.
3. Exactly 32 raw bytes of the source SHA-256 digest.

Text fields contain 1–256 printable ASCII bytes without whitespace. Versions use
the package library's numeric version parser. Git commits are exactly 40 lowercase
hex characters (SHA-1 object identity, not the source-integrity digest). Compiler
The source language is `luce` or `luce-base`. Rejected encoding leaves output unchanged.

`pkg.verify_release` requires an independently trusted 1952-byte public key and
3309-byte signature. It verifies the exact encoded metadata, hashes the supplied
source bytes, and compares their digest. The source bytes must be the canonical
package artifact; Git tree/archive canonicalization is still separate work.
No untrusted embedded key is accepted as a trust root, and there is no classical
signature fallback. Invalid metadata raises an error; failed signature/digest
checks return false. Callers must treat both as rejection.

`pkg.decode_release` parses downloaded metadata with bounded lengths and exact
EOF, validates its fields through the canonical encoder, and rejects unknown
schemas, truncation, trailing bytes and noncanonical representations. Returned
strings/digest borrow the input: keep that buffer alive and unchanged through
verification. Parsing alone does not authenticate it. The decoder is tested at
every truncation boundary of a complete fixture.

This API does not fetch keys, validate origin ownership or toolchain constraint
semantics, enforce expiry/rollback protection, authorize publishers, unpack an
archive, or install files. Registry-root roles and account-key custody remain
separate required layers. Tests use disposable deterministic keys only.

## LRS2 dependency binding

`pkg.encode_release_v2` emits ASCII `LRS2` and little-endian schema2. It writes
the same first six fields as LRS1, followed by a seventh compiler package identity
field. The 32-byte source digest follows, then a little-endian u16 dependency
count. Each dependency contains two u16-length-prefixed strings: its registry
coordinate and exact-or-caret numeric-semver requirement.

The compiler package identity is a lowercase identifier of at most128 bytes.
Dependencies use the same canonical owner/package syntax as registry routes,
are strictly sorted by coordinate, unique, same-origin by construction, and may
not directly name the release itself. Limits are128 entries,97 bytes per
coordinate,63 bytes per requirement,20,992 dependency bytes and24,576 total
metadata bytes. Exact framing is part of the signature. `dependency_at` exposes
borrowed entries only after the complete metadata has passed canonical decoding.

LRS1 remains accepted for existing immutable releases. Its absent package identity
and dependency declaration must not be interpreted as a dependency-free LRS2
release. New dependency-aware publication and resolution require schema2. Neither
schema proves that a repository's `luce.toml` matches the signed declaration;
publication must perform that independent source/manifest check.
