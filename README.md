# luce-pkg

Package definitions for Luce, written in Luce Base. MIT OR Apache-2.0.

The `pkg` export decodes `package.prisma`, the one authored project file, into a
`Definition`: identity, owner, version, kind (`package`, `application` or `tool`),
language, entry, description and README, dependencies, exports, tasks, native
inputs and the desktop bundle. Unknown elements and properties are errors, and each
kind decides what else the file must and must not say. The format is described in
[docs/PACKAGE_PRISMA.md](docs/PACKAGE_PRISMA.md).

It also parses numeric `major.minor.patch` versions and computes the SHA-256 that
`luc.lock` records for a release and the registry lists beside it.

Consumers: [luce-luc](https://github.com/dymokomi/luce-luc) and
[luce-pkg-server](https://github.com/dymokomi/luce-pkg-server).

## Test

```sh
python3 tools/bootstrap.py
python3 tests/run.py          # every compiler mode; --mode native0 for one
```
