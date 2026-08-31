# Changelog

All notable changes to the shared CI actions are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Consumers (tanh-lib, anira) pin a release tag instead of `@main`; a breaking
change here updates the consumers in the same motion.

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
