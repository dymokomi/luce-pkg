# Provenance

Original Luce Base package identity, caret resolution and `luce.lock` handling,
Copyright 2026 Dy Mokomi, MIT OR Apache-2.0. No foreign package solver or
registry client is included or linked.

SemVer comparison follows the numeric major.minor.patch subset of
https://semver.org/spec/v2.0.0.html. Prerelease and build metadata are rejected
in v1. Caret matching follows Cargo's 0.x minor-lock rule.

SHA-256 source digests use `luce-crypto`. Compilers do not read `luce.lock`.
Build/test/bootstrap scaffolding follows the MIT OR Apache-2.0 `luce-crypto`
and `luce-compress` patterns by the same author.
