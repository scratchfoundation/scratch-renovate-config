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

### `js-lib.json` and `js-app.json`

These build on the `default.json` configuration and enables pinning according to the recommendations found
[here](https://docs.renovatebot.com/dependency-pinning/). In short, `js-lib` enables pinning of only `devDependencies`
and `js-app` enables pinning of all except `peerDependencies`. These are intended to be the Scratch versions of
Renovate's built-in `config:js-lib` and `config:js-app` presets.

### `conservative.json`

This legacy configuration enables automatic merging of major and minor releases for only LLK dependencies.

## Contributing

Before submitting a pull request, please validate your changes:

```sh
npx --package=renovate@latest renovate-config-validator *.json
```
