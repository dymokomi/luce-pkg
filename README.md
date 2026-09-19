# luce-pkg

Native Luce Base package identity, caret resolution and `luc.lock` handling.
MIT OR Apache-2.0. Compilers still read `luce.toml`; this package owns the
lockfile. No foreign solver.

Experimental. Not a complete installer, TUF client or registry.

The current encoder rejects quotes, backslashes and control bytes in literal
fields to prevent TOML injection; invalid input leaves the output buffer unchanged.
It is a restricted v1 encoder, not a general TOML parser. `lock_origin` is still
a prototype extractor and must not be used as a trust decision for downloaded
lockfiles. Full lock decoding, signature verification and remote installation
remain required before integration into `luc`.

```sh
python3 tools/bootstrap.py
python3 tests/run.py
```
