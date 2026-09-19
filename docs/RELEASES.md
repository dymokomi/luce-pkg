# LRS1 release binding

`pkg.encode_release` produces the message signed with ML-DSA-65. Byte layout:

1. ASCII `LRS1`, followed by little-endian u16 schema `1`.
2. Six text fields in order: origin, package coordinate, version, Git commit,
   compiler identity, toolchain constraint. Each is u16 little-endian byte length
   followed by that many bytes. No terminators or padding.
3. Exactly 32 raw bytes of the source SHA-256 digest.

Text fields contain 1–256 printable ASCII bytes without whitespace. Versions use
the package library's numeric version parser. Git commits are exactly 40 lowercase
hex characters (SHA-1 object identity, not the source-integrity digest). Compiler
identity is `luce` or `luce-base`. Rejected encoding leaves output unchanged.

`pkg.verify_release` requires an independently trusted 1952-byte public key and
3309-byte signature. It verifies the exact encoded metadata, hashes the supplied
source bytes, and compares their digest. The source bytes must be the canonical
package artifact; Git tree/archive canonicalization is still separate work.
No untrusted embedded key is accepted as a trust root, and there is no classical
signature fallback. Invalid metadata raises an error; failed signature/digest
checks return false. Callers must treat both as rejection.

This API does not fetch keys, validate origin ownership or toolchain constraint
semantics, enforce expiry/rollback protection, authorize publishers, unpack an
archive, or install files. Registry-root roles and account-key custody remain
separate required layers. Tests use disposable deterministic keys only.
