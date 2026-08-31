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
