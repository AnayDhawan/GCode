# OS support matrix

GCode is pure Python and its dependencies (`questionary`, `prompt_toolkit`,
`rich`, `langchain`) are all cross-platform, so the CLI, `/models` menu,
history, and the pure-Python `grep` fallback work the same everywhere. The
platform differences are in the tools that shell out: `execute_bash` and the
git tools.

| Platform | CI coverage | `execute_bash` | git tools | `grep` |
| --- | --- | --- | --- | --- |
| Linux | Full matrix (Python 3.10-3.13) | Runs via `/bin/sh`, bash syntax works | Works if `git` is on `PATH` | Uses system `grep` |
| macOS | Smoke run (Python 3.13) | Runs via `/bin/sh`, bash syntax works | Works if `git` is on `PATH` | Uses system `grep` (BSD grep; see note below) |
| Windows (native) | Smoke run (Python 3.13) | Runs via `cmd.exe`, **bash syntax fails** ([#54](https://github.com/shauryagangrade/GCode/issues/54)) | Needs Git for Windows | Falls back to the pure-Python search |
| Windows (WSL2 / Git Bash) | Not separately covered in CI (Linux job is the proxy) | Runs via `bash`, works as documented | Works | Uses system `grep` |

See [docs/windows.md](windows.md) for the full native/WSL2/Git Bash setup
guide and known limitations.

## Why macOS and Windows only get a smoke run

The Linux job already runs the full test suite across every supported Python
version (3.10-3.13). macOS and Windows add the same suite on the newest
supported version only, to catch OS-specific regressions (path handling,
`shell=True` behavior, encoding) without tripling CI cost for coverage that
duplicates what the Linux matrix already proves. Widen it if a
platform-specific bug slips through on an untested Python version.

## Note: macOS ships BSD grep, not GNU grep

The `grep` tool passes GNU-style flags (`-rnIH`, `--include=<glob>`). GNU
grep and modern BSD grep (macOS's default, and the one on the CI
`macos-latest` runner) both accept this flag set, but if you hit a
`grep`-shaped failure specifically on macOS, this divergence is the first
place to look — see the differential tests in
`tests/test_grep_differential.py`, which run against whatever `grep` binary
is on `PATH`.
