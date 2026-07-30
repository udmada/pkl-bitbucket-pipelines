[![GitHub Release](https://img.shields.io/github/v/release/udmada/pkl-bitbucket-pipelines?include_prereleases)](https://github.com/udmada/pkl-bitbucket-pipelines/releases/latest)
[![GitHub License](https://img.shields.io/github/license/udmada/pkl-bitbucket-pipelines)](https://github.com/udmada/pkl-bitbucket-pipelines/blob/master/LICENSE)
[![CI](https://github.com/udmada/pkl-bitbucket-pipelines/actions/workflows/ci.yml/badge.svg)](https://github.com/udmada/pkl-bitbucket-pipelines/actions/workflows/ci.yml)

# <img src="assets/pkl.svg" alt="Pkl" width="40"/> pkl-bitbucket-pipelines <img src="assets/bitbucket.svg" alt="Bitbucket" width="48"/>

A [Pkl](https://pkl-lang.org/) template for writing Bitbucket Pipelines workflows.

Describe your pipeline as typed Pkl, get a validated `bitbucket-pipelines.yml` out. Mistakes that
Bitbucket would otherwise only surface on a push — a misspelled property, a `max-time` over the
720-minute ceiling, an ECR image with half its credentials, a `retry` count on an `ignore`
strategy — fail at evaluation time instead.

Modelled on the
[Bitbucket Pipelines configuration reference](https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/),
and structured after Apple's [`com.circleci.v2`](https://github.com/apple/pkl-pantry/tree/main/packages/com.circleci.v2)
package in pkl-pantry and StefMa's [`pkl-gha`](https://github.com/StefMa/pkl-gha) package on GitHub.

## Install

### Amend the template directly

The simplest form. Point `amends` at the package and write your pipeline:

```pkl
amends "package://pkg.pkl-lang.org/github.com/udmada/pkl-bitbucket-pipelines/com.atlassian.bitbucket.pipelines@1.0.0#/Config.pkl"

image = "node:24"

pipelines {
  `default` {
    new Step {
      name = "Build and test"
      caches { "node" }
      script {
        "npm ci"
        "npm test"
      }
      artifacts { "dist/**" }
    }
  }
}
```

### As a project dependency

Preferable in a repo that already has a `PklProject`, because the version is pinned once in
`PklProject.deps.json` rather than repeated in every module:

```pkl
// PklProject
amends "pkl:Project"

dependencies {
  ["com.atlassian.bitbucket.pipelines"] {
    uri = "package://pkg.pkl-lang.org/github.com/udmada/pkl-bitbucket-pipelines/com.atlassian.bitbucket.pipelines@1.0.0"
  }
}
```

Run `pkl project resolve`, commit the resulting `PklProject.deps.json`, then use the short
`@`-notation:

```pkl
amends "@com.atlassian.bitbucket.pipelines/Config.pkl"
```

### From a local checkout

For working on the schema and a consuming config at the same time:

```pkl
dependencies {
  ["com.atlassian.bitbucket.pipelines"] = import("../pkl-bitbucket-pipelines/PklProject")
}
```

## Render

```console
pkl eval -f yaml -o bitbucket-pipelines.yml MyPipeline.pkl
```

Commit the generated `bitbucket-pipelines.yml` — Bitbucket reads that file, not your Pkl. A good
habit is a pipeline step that regenerates it and fails if the result differs from what was
committed, the same trick this repo uses for its own GitHub workflows.

Note that `default` and `import` are Pkl keywords, so they need backticks: `` `default` ``,
`` `import` ``. The same goes for any hyphenated property, such as `` `max-time` `` and
`` `pull-requests` ``.

## What's covered

| Area | Types |
| --- | --- |
| Root | `image`, `clone`, `options`, `export`, `definitions`, `pipelines`, `triggers` |
| Start conditions | `PipelineDefinitions` — `default`, `branches`, `pull-requests`, `tags`, `custom` |
| Pipeline entries | `Step`, `FinalStep`, `ChildPipelineStep`, `Stage`, `Parallel`, `ParallelSteps`, `Variables` |
| Scripts | `ScriptItem`, `Pipe` |
| Artifacts | `Artifacts`, `ArtifactsSpec`, `ArtifactUpload`, `ChildPipelineArtifacts` |
| Flow control | `Condition`, `Changesets`, `OnFail`, `Trigger` |
| Event triggers | `Triggers`, `TriggerCondition` — including `glob`, `changesetInclude`, `changesetExclude` |
| Definitions | `Cache`, `CacheKey`, `Service`, `imports`, exported `pipelines` |
| Images | `Image`, `ImageSpec`, `AwsAuth` |
| Runtime | `GlobalOptions`, `Runtime`, `CloudRuntime` (including Runtime v3), `Size`, `MaxTime` |

Every property carries the documentation from the Atlassian reference, so editors with
[Pkl language support](https://pkl-lang.org/main/current/tools.html) show allowed values,
defaults, and plan restrictions inline. The same doc comments are published as browsable API
documentation at **[udmada.github.io/pkl-bitbucket-pipelines](https://udmada.github.io/pkl-bitbucket-pipelines/)**.

### YAML anchors are deliberately omitted

`definitions.steps` exists only to host YAML anchors, which work around YAML's lack of
abstraction. Pkl has abstraction, so use it — a `local` function returns a step you can
parameterize per call site, which anchors cannot do:

```pkl
local function testShard(shard: Int, of: Int): Step = new {
  name = "Test \(shard)/\(of)"
  script { "npm test -- --shard=\(shard)/\(of)" }
}

pipelines {
  `default` {
    new Parallel {
      steps {
        testShard(1, 3)
        testShard(2, 3)
        testShard(3, 3)
      }
    }
  }
}
```

Two things to know when reusing steps this way:

- Amending a step **appends** to its `Listing` properties. `(buildStep) { script { "extra" } }`
  adds a command; to replace the script outright, assign `script = new { … }`.
- Inside `new Step { … }`, an unqualified `name` resolves to the step's own `name` property, not
  to an enclosing function parameter of the same name. Name parameters distinctly — `service`,
  not `name` — or you'll get a stack overflow.

## Examples

| File | Shows |
| --- | --- |
| [`basic.pkl`](examples/basic.pkl) | The smallest useful pipeline |
| [`reusable_steps.pkl`](examples/reusable_steps.pkl) | Parameterized steps in place of YAML anchors |
| [`deployment_pipeline.pkl`](examples/deployment_pipeline.pkl) | Deployment stages, manual gates, retries, a final step |
| [`monorepo_triggers.pkl`](examples/monorepo_triggers.pkl) | Changeset-driven `triggers` building only what changed |
| [`child_pipelines.pkl`](examples/child_pipelines.pkl) | `type: pipeline` steps, input variables, artifacts across the boundary |
| [`private_images_and_services.pkl`](examples/private_images_and_services.pkl) | ECR via OIDC, service containers, Runtime v3 buildx, self-hosted runners |
| [`shared_pipelines.pkl`](examples/shared_pipelines.pkl) | `export`, `definitions.imports`, and importing a shared pipeline |

## Development

Tasks are defined in [`mise.toml`](mise.toml):

```console
mise run test          # snapshot-test every example
mise run test:record   # re-record after an intended schema change
mise run package       # build the publishable package into .out/
mise run pkl:gen       # regenerate .github/workflows/ from .github/pkl/
```

This repo's own GitHub Actions workflows are written in Pkl under
[`.github/pkl/`](.github/pkl) using pkl-pantry's `com.github.actions`, and generated into
`.github/workflows/`. Actions are pinned to git SHAs recorded in
`.github/workflows/__lockfile__.yml`, which dependabot keeps current. Edit the Pkl, never the
generated YAML — CI fails if the two disagree.

### API documentation

The [Docs workflow](.github/pkl/docs.pkl) publishes pkldoc output to GitHub Pages on every push to
`master`. Note that pkldoc is **not** part of the native `pkl` CLI, which has no `doc` subcommand —
it lives in the JVM `pkl-tools` jar, so the workflow fetches that from Maven Central pinned to the
same version the rest of CI uses. To generate the docs locally:

```console
curl -sSLf -o /tmp/pkl-tools.jar \
  https://repo1.maven.org/maven2/org/pkl-lang/pkl-tools/0.32.1/pkl-tools-0.32.1.jar
java -cp /tmp/pkl-tools.jar org.pkl.doc.Main doc-package-info.pkl Config.pkl -o .out/doc
```

`doc-package-info.pkl` must be passed alongside the modules — pkldoc refuses to run without it,
and it supplies the package name, version, and import URI the pages are built around.

## Releasing

Publishing is driven entirely by the tag name, because that is what makes a package resolvable.
`pkg.pkl-lang.org` is a redirector: it rewrites

```
package://pkg.pkl-lang.org/github.com/udmada/pkl-bitbucket-pipelines/com.atlassian.bitbucket.pipelines@1.0.0
```

to

```
https://github.com/udmada/pkl-bitbucket-pipelines/releases/download/com.atlassian.bitbucket.pipelines@1.0.0/com.atlassian.bitbucket.pipelines@1.0.0
```

which must be the package metadata file, sitting alongside the `.zip` it names. So a release has
to be tagged exactly `<name>@<version>` and carry all four artifacts that `pkl project package`
emits.

To cut a release:

1. Bump `package.version` in [`PklProject`](PklProject) and `version` in
   [`doc-package-info.pkl`](doc-package-info.pkl).
2. Commit, then tag and push:

   ```console
   git tag com.atlassian.bitbucket.pipelines@1.0.1
   git push origin com.atlassian.bitbucket.pipelines@1.0.1
   ```

The [Release workflow](.github/pkl/release.pkl) packages the module, verifies the tag matches
`package.version`, and publishes the artifacts to a GitHub release.

Note that the redirector only understands `github.com` paths — `bitbucket.org` and `gitlab.com`
return 404. A package released elsewhere needs `package.baseUri` pointed at a host you serve the
metadata file from yourself.

## Why this package has no dependencies

`Config.pkl` deliberately depends on nothing but the Pkl standard library, so consumers inherit no
transitive dependencies — the same choice `com.circleci.v2` makes. In particular, none of the
`pkl.experimental.*` pantry packages are used:

| Package | Why not |
| --- | --- |
| `pkl.experimental.deepToTyped` | Would only help a YAML-to-Pkl importer, and it cannot see through this schema's discriminated wrappers: a parsed `- step: {…}` has no structural match on `Step`, so it silently falls through to defaults. |
| `pkl.experimental.structuredRead` | Reads env vars and external resources at evaluation time, which would make the generated YAML depend on where it was generated. A committed pipeline file should be reproducible. |
| `pkl.experimental.syntax` | Emits Pkl source. Useful for a migration tool, not for a schema. |
| `pkl.experimental.uri` | Image references and import-source slugs are Bitbucket's own string grammars, not URIs; the existing regex constraints cover them. |

The `experimental` prefix is also a stability contract: a published schema that depends on those
would break its consumers whenever they churn. They are reasonable choices for a dev-only tool in
this repo (excluded from the package), which is exactly how `com.github.actions` is used for the
workflows under `.github/pkl/`.

## Caveats

A few limits in the Atlassian reference are prose-only and are documented on the relevant types
rather than enforced, because they depend on context this schema cannot see:

- The 100-step ceiling per pipeline, and the 10-or-100-step ceiling per stage (plan-dependent).
- A `FinalStep` must be last, and at least one non-final step must precede it.
- Manual steps and stages cannot come first in a pipeline.
- Steps in a `Stage` cannot override a property set on the stage.
- Sizes above `2x`, `arch = "arm"`, and configuration sharing require a paid or Premium plan.

`Cache.path` is required here, though the reference marks it optional; a keyed cache with no path
has nothing to store. Use the string shorthand — `["name"] = "some/path"` — for a cache that
needs no key.

## License

[Apache-2.0](LICENSE)

This project is not affiliated with or endorsed by Atlassian or Apple. The Bitbucket mark in
[`assets/bitbucket.svg`](assets/bitbucket.svg) comes from Atlassian's
[official logo pack](https://atlassian.design/resources/logo-library) and the Pkl mark in
[`assets/pkl.svg`](assets/pkl.svg) from [pkl-lang.org](https://pkl-lang.org/); both remain
trademarks of their respective owners and are used here only to identify what this package targets.
