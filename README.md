# scratch-renovate-config

Scratch's shared configuration files for [Renovate](https://docs.renovatebot.com/)

While Renovate supports JSON5 in some contexts, the configuration files in this repository are in strict JSON format.
See renovatebot/renovate#7674 for more information.

## Available configurations

Use the `extends` array within your Renovate configuration file to extend one of these configurations. For example:
`extends: ["github>scratch-renovate-config"]`

Name | File
--- | ---
`github>scratchfoundation/scratch-renovate-config:base` | `base.json`
`github>scratchfoundation/scratch-renovate-config` | `default.json`
`github>scratchfoundation/scratch-renovate-config:js-lib` | `js-lib.json`
`github>scratchfoundation/scratch-renovate-config:js-lib-bundled` | `js-lib-bundled.json`
`github>scratchfoundation/scratch-renovate-config:js-app` | `js-app.json`
`github>scratchfoundation/scratch-renovate-config:conservative` | `conservative.json`

### `base.json`

This applies basic configuration without any automatic merge rules. It's intended to be used by the other
configurations in this repository, not by itself.

The common configuration in this file includes:

* set time zone to Scratch time (`America/New_York`)
* remove concurrent PR limit
* enable "semantic commit" rules:
  * use `fix` for `dependencies`
  * use `chore` for `devDependencies`
* require 3 "stability days" before automatically merging non-LLK dependencies
  * this configuration does not enable automatic merges but does preconfigure this setting
* label all Renovate PRs with `dependencies`
  * add the `security` label for any PR associated with a GitHub Security Vulnerability
* separate major updates from minor/patch updates: if both are available, open two separate PRs

### `default.json`

This enables automatic merging of minor and patch releases. Dependencies outside of the LLK organization are subject
to the "stability days" settings from `base.json`. This can be used directly, but the `js-lib` and `js-app`
configurations may be more appropriate.

### `js-*.json`

These build on the `default.json` configuration and enables pinning according to the recommendations found
[here](https://docs.renovatebot.com/dependency-pinning/). In short, `js-lib` enables pinning of only `devDependencies`
and `js-app` enables pinning of all except `peerDependencies`. These are intended to be the Scratch versions of
Renovate's built-in `config:js-lib` and `config:js-app` presets.

The `js-lib-bundled` configuration is a variant of `js-lib` designed for libraries that use a bundler (`webpack`,
etc.) to build their npm package. It treats `lockFileMaintenance` PRs as `fix` instead of `chore`. For rationale, see
the "About Pinning" section below.

### `conservative.json`

This legacy configuration enables automatic merging of major and minor releases for only LLK dependencies.

## About Pinning

_See [Renovate's documentation](https://docs.renovatebot.com/dependency-pinning/) for more information._

The `js-lib` and `js-app` configurations enable pinning of dependencies according to Renovate's recommendations:

* `devDependencies` are pinned to exact versions everywhere
* `dependencies` are pinned to exact versions for `js-app` but use `^x.y.z` in repos using `js-lib`

**Together, this means that non-dev dependencies of our libraries are pinned at the app level only.**

More specifically, suppose `scratch-vm` uses a library `foo` as a `dependency` (not `devDependency`), and `foo`
updates from `1.0.0` to `1.1.0`. Suppose neither `scratch-gui` nor `scratch-www` use `foo` as a direct dependency.

1. `scratch-vm`, where `foo` is listed as `^1.0.0`, will receive a `lockFileMaintenance` PR to update `foo`.
   * For a "normal" npm module, this PR would be unable to affect user-visible behavior, because it will not affect
     `package.json` or any other file.
   * Since `scratch-vm` uses `webpack`, dependencies like `foo` are likely to be included in the build output, thus this
     PR _could_ cause user-visible behavior changes.
     * If `scratch-vm`'s build output is used by an app, then `scratch-vm`'s `lockFileMaintenance` PR could cause
       user-visible behavior changes in that app, implying that this _should_ cause a `scratch-vm` release.
     * If the app uses `scratch-vm`'s source instead of its build output, then `scratch-vm`'s `lockFileMaintenance` PR
       cannot cause user-visible behavior changes in the app, implying that this should _not_ cause a `scratch-vm`
       release.
2. `scratch-www` will also receive a `lockFileMaintenance` PR to update `foo`.
   * `foo` is not listed in `scratch-www`'s `package.json`, so this PR won't affect that file.
   * If `scratch-www` uses `scratch-vm`'s build output, then `scratch-www`'s `lockFileMaintenance` PR will not cause
     user-visible behavior changes in `scratch-www`. The app would receive any user-facing impact from the `foo`
     update when `scratch-www` updates its `scratch-vm` dependency.
   * If `scratch-www` uses `scratch-vm`'s source, then `scratch-www`'s `lockFileMaintenance` PR will cause user-visible
     behavior changes in `scratch-www`. The app could receive some user-facing impact from the `foo` update here and
     some from the `scratch-vm` update, or only one or the other.

In other words, by using `webpack` (or any bundler) at the library level, we have made it difficult to predict the
effects of a `lockFileMaintenance` PR and blurred the meaning of `dependencies` vs. `devDependencies`. Ideally, we
should use `webpack` only at the app level; combined with Renovate's pinning recommendations, this would mean that
library `lockFileMaintenance` changes could never affect user-visible behavior at the app level. Only app
`lockFileMaintenance` changes could affect user-visible behavior. For now, marking `lockFileMaintenance` PRs as `fix`
seems the safest option.

## Contributing

Before submitting a pull request, please validate your changes:

```sh
npx --package=renovate@latest renovate-config-validator *.json
```
