# Changelog

All notable changes to this package are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the package adheres to
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Changed

- **Breaking:** `CloudRuntime.version` accepts only the string literals `"2"` and `"3"`, matching
  the documented YAML form. Previously an `Int` was also accepted and rendered unquoted.
- Pipeline element ordering rules are now enforced at evaluation time instead of only being
  documented: a `FinalStep` must be the last element and follow at least one ordinary step, a
  `Variables` element may only come first, and the first step or stage of a pipeline cannot be
  `manual`.

### Added

- Negative tests under `tests/invalid/`: configurations that must fail evaluation, guarding the
  schema's constraints against being accidentally loosened. Run with `mise run test:invalid`,
  included in `mise run test`.
- `mise.lock` is committed, so local and CI toolchain installs resolve identically.

## [1.1.0] - 2026-07-30

### Added

- API documentation generated with pkldoc and published to GitHub Pages, with a `mise run doc`
  task to build it locally.
- `mise` as the single source of truth for the toolchain, replacing the setup-pkl and setup-java
  CI actions; `mise run version:set` bumps the package version everywhere at once.
- A version-consistency test: every `com.atlassian.bitbucket.pipelines@<version>` reference in
  the README and `Config.pkl` must match `PklProject`, so install snippets cannot go stale.
- README, logos, and GitHub Actions workflows — themselves authored in Pkl and drift-checked
  in CI.

## [1.0.0] - 2026-07-30

### Added

- Initial release: a typed Pkl schema for `bitbucket-pipelines.yml`, modelled on the Bitbucket
  Pipelines configuration reference — pipelines, steps, stages, parallel groups, child
  pipelines, triggers, artifacts, caches, services, images, and runtime options, with
  Bitbucket's limits enforced as type constraints.
- Seven examples covering the schema surface, each rendered and snapshot-tested.

[Unreleased]: https://github.com/udmada/pkl-bitbucket-pipelines/compare/com.atlassian.bitbucket.pipelines@1.1.0...HEAD
[1.1.0]: https://github.com/udmada/pkl-bitbucket-pipelines/compare/com.atlassian.bitbucket.pipelines@1.0.0...com.atlassian.bitbucket.pipelines@1.1.0
[1.0.0]: https://github.com/udmada/pkl-bitbucket-pipelines/releases/tag/com.atlassian.bitbucket.pipelines@1.0.0
