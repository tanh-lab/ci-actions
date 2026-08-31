# Changelog

All notable changes to the shared CI actions are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Consumers (tanh-lib, anira) pin a release tag instead of `@main`; a breaking
change here updates the consumers in the same motion.

## [0.2.2] - 2026-08-31

### Fixed

- `clang-tidy-check`'s `FILES` input reaches the script through the environment
  instead of inline substitution — a multi-line file list broke the shell
  syntax (`syntax error near unexpected token`), and inline substitution of
  caller-provided text was an injection hazard besides.

## [0.2.1] - 2026-08-31

### Fixed

- `setup-cpp-build-tools` matches every `Linux-*` matrix name, not only
  `Linux-x86_64*` — arm64 Linux legs (anira: `ubuntu-24.04-arm`) previously fell
  through to the unknown-platform branch and got no ninja/clang (apt.llvm.org
  serves arm64 packages).

## [0.2.0] - 2026-08-31

### Added

- `cmake-build` appends `CMAKE_BUILD_ARGS` in preset mode too (previously
  manual-mode-only), so one preset can serve several configurations — e.g.
  anira's backend sets ride on platform presets instead of a preset per
  combination.

- `clang-tidy-check` gains an optional `FILES` input for diff-based runs: pass
  a changed-files list (e.g. a PR's) and only the entries inside the `SOURCES`
  scope are analysed — an empty intersection passes. Full-sweep behavior when
  omitted is unchanged. A full anira sweep is ~6 min of analysis; most PRs
  touch a handful of files.

## [0.1.0] - 2026-08-31

First tagged release — the baseline consumers pin. Contains the seven actions
as grown on `main`: `setup-cpp-build-tools`, `cmake-build`, `cmake-test`,
`cmake-test-android`, `cmake-test-ios-simulator`, `clang-format-check`,
`clang-tidy-check`. All actions detect the platform via the calling job's
`matrix.name` (see README).

### Fixed

- `cmake-build` exports `ACTIONS_CACHE_SERVICE_V2=on` before starting the
  sccache server. Without it, sccache's GHA cache client falls back to the
  retired v1 cache API — every read 404s and every write fails with "Cannot
  write to read-only storage", a permanent 0% hit rate for every consumer.

### Added

- `cmake-build` keeps the configure output in `configure.log` (both preset and
  manual mode), so calling workflows can assert on it — e.g. anira greps for
  its backend auto-disable warnings.
- `cmake-test` gains an optional `CTEST_ARGS` input, passed through in both
  modes — e.g. `-j 4` to run the per-case gtest entries in parallel, or `-R`
  to filter.
