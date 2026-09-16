# luce-pkg

Native Luce Base package identity, caret resolution and `luce.lock` handling.
MIT OR Apache-2.0. Compilers still read `luce.toml`; this package owns the
lockfile. No foreign solver.

Experimental. Not a complete installer, TUF client or registry.

```sh
python3 tools/bootstrap.py
python3 tests/run.py
```
