# ci-actions

Shared GitHub Actions for tanh-lab C++ CMake projects.

## Actions

| Action | Description |
| --- | --- |
| `setup-cpp-build-tools` | Install platform-specific C++ build tools (clang, ninja, etc.) |
| `cmake-build` | Configure and build a CMake project (supports presets and manual mode) |
| `cmake-test` | Run CTest (supports presets and manual mode) |
| `clang-format-check` | Check C++ source formatting with clang-format (fails on violations) |
| `clang-tidy-check` | Run clang-tidy on C++ sources (fails on any warning) |
| `changed-files` | List the files a pull request changes, from git alone (no token), and say whether a check should sweep everything instead |

## Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: tanh-lab/ci-actions/setup-cpp-build-tools@main
    with:
      CLANG_VERSION: "20"  # optional, default: 20

  - uses: tanh-lab/ci-actions/cmake-build@main
    with:
      PRESET: desktop-debug  # or use BUILD_TYPE for manual mode

  - uses: tanh-lab/ci-actions/cmake-test@main
    with:
      PRESET: desktop-debug

  - uses: tanh-lab/ci-actions/clang-format-check@main
    with:
      SOURCES: "src/ include/"

  - uses: tanh-lab/ci-actions/clang-tidy-check@main
    with:
      SOURCES: "src/"
      PRESET: desktop-debug
```

## Checking only what a pull request changes

`clang-tidy-check` sweeps `SOURCES` by default. With `CHANGED_FILES_ONLY: "true"`
a pull_request run checks only the `.cpp` files the pull request changes, and
every other event still sweeps (the merge queue; a `workflow_dispatch` run
sweeps a branch by hand):

```yaml
  - uses: tanh-lab/ci-actions/clang-tidy-check@v0.3.13
    with:
      SOURCES: "src/ test/"
      PRESET: desktop-debug
      CHANGED_FILES_ONLY: "true"
      # A build or tool config change re-judges every file: sweep.
      FULL_SWEEP_PATTERN: '(^|/)CMakeLists\.txt$|\.cmake$|^CMakePresets\.json$|^\.clang-tidy$'
```

The list comes from git: on a pull_request event `actions/checkout` leaves HEAD
on GitHub's merge commit, and the diff against its first parent is what the pull
request changes. No token and no API call; `fetch-depth: 2` saves one fetch, the
default depth works too. Every doubt sweeps: another event, a path
matching the pattern, a checkout that is not that merge commit.

The trade: a header change does not widen the set, so what it breaks in a file
the pull request did not touch shows in the next sweep, not on the pull request.
Keep a sweeping run as the enforcement point (the merge queue).

`changed-files` is the same logic as its own action, for any other check:

```yaml
  - uses: tanh-lab/ci-actions/changed-files@v0.3.13
    id: changed
    with:
      FULL_SWEEP_PATTERN: '^\.clang-format$'
  # steps.changed.outputs.sweep  'true' | 'false'
  # steps.changed.outputs.files  newline-separated paths, deleted ones included
  # steps.changed.outputs.reason one line for the log
```

## Matrix naming convention

Actions use `matrix.name` for platform detection:

- `Linux-x86_64`
- `macOS-x86_64`
- `macOS-arm64`
- `Windows-x86_64`

## Versioning

Consumers pin a release tag (`uses: tanh-lab/ci-actions/<action>@v0.1.0`)
instead of `@main`. Changes are recorded in [CHANGELOG.md](CHANGELOG.md);
a breaking change updates the consumers (tanh-lib, anira) in the same motion.
