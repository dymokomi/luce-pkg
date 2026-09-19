# luce-pkg

Native Luce Base package identity, caret resolution and `luc.lock` handling.
MIT OR Apache-2.0. Compilers still read `luce.toml`; this package owns the
lockfile. No foreign solver.

Experimental. Not a complete installer, TUF client or registry.

The current encoder rejects quotes, backslashes and control bytes in literal
fields to prevent TOML injection; invalid input leaves the output buffer unchanged.
It uses a restricted v1 TOML profile, not general TOML syntax. `decode_lock`
validates the full document and returns a closeable owner of all decoded strings;
`lock_origin` validates the same document but returns a string borrowing its input.
Headers must precede packages; schema 1 and one origin are required. Each package
requires name, numeric version, digest and compiler exactly once. Duplicate package
names, unknown keys/tables, escapes and incomplete entries are rejected. Blank
lines, full-line comments, horizontal whitespace and CRLF are supported. Limits
are 1 MiB of input and 4096 packages. Digest/identity policy, release signature
verification and remote installation are separate requirements, not established
by successful parsing.

```sh
python3 tools/bootstrap.py
python3 tests/run.py
```
