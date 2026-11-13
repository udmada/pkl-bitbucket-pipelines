# Changelog

All notable changes to this package are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the package adheres to
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Fixed

- A `ChildPipelineStep` without `input-variables` failed to evaluate with "Cannot find property
  `length`". The property is optional again, as documented. Every earlier release is affected.

### Changed

- **Breaking:** configurations that Bitbucket rejects on push, and that this schema used to
  render, now fail at evaluation time:
  - an empty pipeline, a pipeline holding only `Variables`, and a `pipelines` block with no
    pipeline in it;
  - a `Variables` element in a pipeline that is not under `custom`;
  - a `condition` on a step inside a `Stage`;
  - more than five `services` on a step;
  - a `manual` step in a `Parallel` group that comes first in its pipeline;
  - both `includePaths` and `excludePaths` in one `Changesets`;
  - an empty `branches`, `pull-requests`, `tags`, or `custom` mapping.
- **Breaking:** names are now cross-checked. A `ChildPipelineStep.custom` or a trigger naming a
  pipeline that is not under `pipelines.custom`, and a step naming a service that is neither
  `docker` nor under `definitions.services`, fail evaluation and report the unknown names.
- Pipeline ordering violations report what is wrong ("A final step must be the last element of
  its pipeline.") instead of printing the whole constraint.
- `Runtime.cloud` is now optional, since a runtime may configure `self-hosted` instead; one of
  the two is still required.

### Added

- Properties from Bitbucket's published JSON schema: root `labels`, `GlobalOptions.runs-on`,
  `BaseStep.auth`, `Runtime.self-hosted` with `SelfHostedRuntime` and its storage types,
  `CloudRuntime.platform` and `CloudRuntime.instance-type`, and the bare-glob form of
  `ArtifactsSpec.upload`.
- `tests/valid/`: edge cases that must keep evaluating, snapshot-tested alongside the examples.
- Each negative test now states the error it expects in an `// expect:` line, so a case passes
  only when it is rejected for the intended reason.

## [1.1.1] - 2026-08-14

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

[1.1.1]: https://github.com/udmada/pkl-bitbucket-pipelines/compare/com.atlassian.bitbucket.pipelines@1.1.0...com.atlassian.bitbucket.pipelines@1.1.1
[1.1.0]: https://github.com/udmada/pkl-bitbucket-pipelines/compare/com.atlassian.bitbucket.pipelines@1.0.0...com.atlassian.bitbucket.pipelines@1.1.0
[1.0.0]: https://github.com/udmada/pkl-bitbucket-pipelines/releases/tag/com.atlassian.bitbucket.pipelines@1.0.0
