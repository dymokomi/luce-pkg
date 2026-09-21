# package.prisma

`package.prisma` is the one authored project file. `luce.toml` is no longer
written by hand: `luc` generates it at the project root (where the compilers look
for it) before every compiler invocation, and it is git-ignored. It is ordinary
Prism text, read with Prism's own decoder. There is no legacy format to support.

```text
#prisma 4.0
def package "luced" {
    str owner = "dymokomi"
    str version = "0.3.0"
    str language = "luce"
    str description = "A text editor written in Luce"
    str readme = "README.md"
    str entry = "src/main.luc"
    str install = "install.luc"

    def dependency "luce_ui" {
        str owner = "dymokomi"
        str version = "^0.4.0"
        str path = "../luce-ui"
    }
    def export "editor" {
        str module = "luced.editor"
    }
    def task "ci" {
        str description = "Everything CI runs"
        str[] depends = ["check", "unit"]
    }
    def task "unit" {
        str cmd = "luc test"
    }
}
```

## Fields

One root `def package "<name>"`. The registry coordinate is `<owner>/<name>`.

| Property | Required | Meaning |
| --- | --- | --- |
| `owner` | yes | Registry account that owns the package. |
| `version` | yes | Numeric `major.minor.patch`. A release tag `v<version>` must match it. |
| `language` | yes | `luce` or `luce-base`. |
| `source` | no | Source directory, default `src`. |
| `description` | no | One line, at most 256 bytes. Shown on the website and in search. |
| `readme` | no | Relative path to a Markdown file rendered on the package page. |
| `entry` | no | Relative path to the program entry. Present: the package is an application. Absent: a library. |
| `install` | no | Relative path to a post-compile install script. Only valid with `entry`. |

Children of the root:

- `def dependency "<name>"` — `<name>` is the dependency's package name, which is
  also the key the compiler requires. `owner` and `version` (a caret requirement
  or an exact version) are required; the registry coordinate is `<owner>/<name>`.
  Optional `path` points at a local checkout that is used instead of the registry
  release, for working across repositories.
- `def export "<import-name>"` with `module`: a public module, as in the
  compiler's `[exports]`.
- `def task "<name>"` with `cmd`, optional `description` and optional
  `str[] depends`: a workflow run by `luc run <name>`.

Names match `[a-z0-9][a-z0-9_-]{0,63}`. Paths are relative, `/`-separated, and
contain no empty, `.` or `..` segment, backslash, colon or NUL. Any other element
kind or property is an error, so a typo is reported rather than ignored. The file
is at most 64 KiB with at most 128 dependencies.

## Install scripts

`luc install owner/name` fetches and builds the application, places the build
output and the package files in a scratch directory, and runs the script with

```text
luce run --sandbox <scratch> install.luc
```

The script is interpreted high-level Luce, confined to the scratch directory with
no network and no child processes. It arranges the files to install under
`<scratch>/out/`. `luc` then copies `out/` to `~/.luce/apps/<name>/<version>/`
and links the executable into `~/.luce/bin`. The script never touches the real
filesystem. An application without `install` gets the default: the built
executable alone. `luc uninstall` removes that directory and the link; it runs no
package code.

## Releases and locks

A release is the source tree at a pushed tag `v<version>`. The registry publishes
it as an immutable tarball with its SHA-256. `luc.lock` records, for every package
in the resolved graph, the coordinate, exact version and that SHA-256; `luc sync`
refuses a download whose digest differs.
