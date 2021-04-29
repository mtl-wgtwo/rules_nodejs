
# Built-in rules

These rules are available without any npm installation,
via the `WORKSPACE` install of the `build_bazel_rules_nodejs` workspace.

This is necessary to bootstrap Bazel to run the package manager to download other rules from NPM.


## node_repositories

**USAGE**

<pre>
node_repositories(<a href="#node_repositories-name">name</a>, <a href="#node_repositories-node_download_auth">node_download_auth</a>, <a href="#node_repositories-node_repositories">node_repositories</a>, <a href="#node_repositories-node_urls">node_urls</a>, <a href="#node_repositories-node_version">node_version</a>,
                  <a href="#node_repositories-package_json">package_json</a>, <a href="#node_repositories-preserve_symlinks">preserve_symlinks</a>, <a href="#node_repositories-repo_mapping">repo_mapping</a>, <a href="#node_repositories-vendored_node">vendored_node</a>, <a href="#node_repositories-vendored_yarn">vendored_yarn</a>,
                  <a href="#node_repositories-yarn_download_auth">yarn_download_auth</a>, <a href="#node_repositories-yarn_repositories">yarn_repositories</a>, <a href="#node_repositories-yarn_urls">yarn_urls</a>, <a href="#node_repositories-yarn_version">yarn_version</a>)
</pre>

To be run in user's WORKSPACE to install rules_nodejs dependencies.

This rule sets up node, npm, and yarn. The versions of these tools can be specified in one of three ways

### Simplest Usage

Specify no explicit versions. This will download and use the latest NodeJS & Yarn that were available when the
version of rules_nodejs you're using was released.
Note that you can skip calling `node_repositories` in your WORKSPACE file - if you later try to `yarn_install` or `npm_install`,
we'll automatically select this simple usage for you.

### Forced version(s)

You can select the version of NodeJS and/or Yarn to download & use by specifying it when you call node_repositories,
using a value that matches a known version (see the default values)

### Using a custom version

You can pass in a custom list of NodeJS and/or Yarn repositories and URLs for node_resositories to use.

#### Custom NodeJS versions

To specify custom NodeJS versions, use the `node_repositories` attribute

```python
node_repositories(
    node_repositories = {
        "10.10.0-darwin_amd64": ("node-v10.10.0-darwin-x64.tar.gz", "node-v10.10.0-darwin-x64", "00b7a8426e076e9bf9d12ba2d571312e833fe962c70afafd10ad3682fdeeaa5e"),
        "10.10.0-linux_amd64": ("node-v10.10.0-linux-x64.tar.xz", "node-v10.10.0-linux-x64", "686d2c7b7698097e67bcd68edc3d6b5d28d81f62436c7cf9e7779d134ec262a9"),
        "10.10.0-windows_amd64": ("node-v10.10.0-win-x64.zip", "node-v10.10.0-win-x64", "70c46e6451798be9d052b700ce5dadccb75cf917f6bf0d6ed54344c856830cfb"),
    },
)
```

These can be mapped to a custom download URL, using `node_urls`

```python
node_repositories(
    node_version = "10.10.0",
    node_repositories = {"10.10.0-darwin_amd64": ("node-v10.10.0-darwin-x64.tar.gz", "node-v10.10.0-darwin-x64", "00b7a8426e076e9bf9d12ba2d571312e833fe962c70afafd10ad3682fdeeaa5e")},
    node_urls = ["https://mycorpproxy/mirror/node/v{version}/{filename}"],
)
```

A Mac client will try to download node from `https://mycorpproxy/mirror/node/v10.10.0/node-v10.10.0-darwin-x64.tar.gz`
and expect that file to have sha256sum `00b7a8426e076e9bf9d12ba2d571312e833fe962c70afafd10ad3682fdeeaa5e`

#### Custom Yarn versions

To specify custom Yarn versions, use the `yarn_repositories` attribute

```python
node_repositories(
    yarn_repositories = {
        "1.12.1": ("yarn-v1.12.1.tar.gz", "yarn-v1.12.1", "09bea8f4ec41e9079fa03093d3b2db7ac5c5331852236d63815f8df42b3ba88d"),
    },
)
```

Like `node_urls`, the `yarn_urls` attribute can be used to provide a list of custom URLs to use to download yarn

```python
node_repositories(
    yarn_repositories = {
        "1.12.1": ("yarn-v1.12.1.tar.gz", "yarn-v1.12.1", "09bea8f4ec41e9079fa03093d3b2db7ac5c5331852236d63815f8df42b3ba88d"),
    },
    yarn_version = "1.12.1",
    yarn_urls = [
        "https://github.com/yarnpkg/yarn/releases/download/v{version}/{filename}",
    ],
)
```

Will download yarn from https://github.com/yarnpkg/yarn/releases/download/v1.2.1/yarn-v1.12.1.tar.gz
and expect the file to have sha256sum `09bea8f4ec41e9079fa03093d3b2db7ac5c5331852236d63815f8df42b3ba88d`.

If you don't use Yarn at all, you can skip downloading it by setting `yarn_urls = []`.

### Using a local version

To avoid downloads, you can check in vendored copies of NodeJS and/or Yarn and set vendored_node and or vendored_yarn
to point to those before calling node_repositories. You can also point to a location where node is installed on your computer,
but we don't recommend this because it leads to version skew between you, your coworkers, and your Continuous Integration environment.
It also ties your build to a single platform, preventing you from cross-compiling into a Linux docker image on Mac for example.

See the [the repositories documentation](repositories.html) for how to use the resulting repositories.

### Manual install

You can optionally pass a `package_json` array to node_repositories. This lets you use Bazel's version of yarn or npm, yet always run the package manager yourself.
This is an advanced scenario you can use in place of the `npm_install` or `yarn_install` rules, but we don't recommend it, and might remove it in the future.

```
load("@build_bazel_rules_nodejs//:index.bzl", "node_repositories")
node_repositories(package_json = ["//:package.json", "//subpkg:package.json"])
```

Running `bazel run @nodejs//:yarn_node_repositories` in this repo would create `/node_modules` and `/subpkg/node_modules`.

Note that the dependency installation scripts will run in each subpackage indicated by the `package_json` attribute.


**ATTRIBUTES**


<h4 id="node_repositories-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this repository.


<h4 id="node_repositories-node_download_auth">node_download_auth</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): auth to use for all url requests
Example: {"type": "basic", "login": "<UserName>", "password": "<Password>" }

Defaults to `{}`

<h4 id="node_repositories-node_repositories">node_repositories</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> List of strings</a>*): Custom list of node repositories to use

A dictionary mapping NodeJS versions to sets of hosts and their corresponding (filename, strip_prefix, sha256) tuples.
You should list a node binary for every platform users have, likely Mac, Windows, and Linux.

By default, if this attribute has no items, we'll use a list of all public NodeJS releases.

Defaults to `{}`

<h4 id="node_repositories-node_urls">node_urls</h4>

(*List of strings*): custom list of URLs to use to download NodeJS

Each entry is a template for downloading a node distribution.

The `{version}` parameter is substituted with the `node_version` attribute,
and `{filename}` with the matching entry from the `node_repositories` attribute.

Defaults to `["https://nodejs.org/dist/v{version}/{filename}"]`

<h4 id="node_repositories-node_version">node_version</h4>

(*String*): the specific version of NodeJS to install or, if vendored_node is specified, the vendored version of node

Defaults to `"12.13.0"`

<h4 id="node_repositories-package_json">package_json</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): (ADVANCED, not recommended)
            a list of labels, which indicate the package.json files that will be installed
            when you manually run the package manager, e.g. with
            `bazel run @nodejs//:yarn_node_repositories` or `bazel run @nodejs//:npm_node_repositories install`.
            If you use bazel-managed dependencies, you should omit this attribute.

Defaults to `[]`

<h4 id="node_repositories-preserve_symlinks">preserve_symlinks</h4>

(*Boolean*): Turn on --node_options=--preserve-symlinks for nodejs_binary and nodejs_test rules.

When this option is turned on, node will preserve the symlinked path for resolves instead of the default
behavior of resolving to the real path. This means that all required files must be in be included in your
runfiles as it prevents the default behavior of potentially resolving outside of the runfiles. For example,
all required files need to be included in your node_modules filegroup. This option is desirable as it gives
a stronger guarantee of hermeticity which is required for remote execution.

Defaults to `True`

<h4 id="node_repositories-repo_mapping">repo_mapping</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>, mandatory*): A dictionary from local repository name to global repository name. This allows controls over workspace dependency resolution for dependencies of this repository.<p>For example, an entry `"@foo": "@bar"` declares that, for any time this repository depends on `@foo` (such as a dependency on `@foo//some:target`, it should actually resolve that dependency within globally-declared `@bar` (`@bar//some:target`).


<h4 id="node_repositories-vendored_node">vendored_node</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>*): the local path to a pre-installed NodeJS runtime.

If set then also set node_version to the version that of node that is vendored.

Defaults to `None`

<h4 id="node_repositories-vendored_yarn">vendored_yarn</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>*): the local path to a pre-installed yarn tool

Defaults to `None`

<h4 id="node_repositories-yarn_download_auth">yarn_download_auth</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): auth to use for all url requests
Example: {"type": "basic", "login": "<UserName>", "password": "<Password>" }

Defaults to `{}`

<h4 id="node_repositories-yarn_repositories">yarn_repositories</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> List of strings</a>*): Custom list of yarn repositories to use.

Dictionary mapping Yarn versions to their corresponding (filename, strip_prefix, sha256) tuples.

By default, if this attribute has no items, we'll use a list of all public NodeJS releases.

Defaults to `{}`

<h4 id="node_repositories-yarn_urls">yarn_urls</h4>

(*List of strings*): custom list of URLs to use to download Yarn

Each entry is a template, similar to the `node_urls` attribute, using `yarn_version` and `yarn_repositories` in the substitutions.

If this list is empty, we won't download yarn at all.

Defaults to `["https://github.com/yarnpkg/yarn/releases/download/v{version}/{filename}"]`

<h4 id="node_repositories-yarn_version">yarn_version</h4>

(*String*): the specific version of Yarn to install

Defaults to `"1.19.1"`


## nodejs_binary

**USAGE**

<pre>
nodejs_binary(<a href="#nodejs_binary-name">name</a>, <a href="#nodejs_binary-chdir">chdir</a>, <a href="#nodejs_binary-configuration_env_vars">configuration_env_vars</a>, <a href="#nodejs_binary-data">data</a>, <a href="#nodejs_binary-default_env_vars">default_env_vars</a>, <a href="#nodejs_binary-entry_point">entry_point</a>, <a href="#nodejs_binary-env">env</a>,
              <a href="#nodejs_binary-link_workspace_root">link_workspace_root</a>, <a href="#nodejs_binary-templated_args">templated_args</a>)
</pre>

Runs some JavaScript code in NodeJS.

**ATTRIBUTES**


<h4 id="nodejs_binary-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this target.


<h4 id="nodejs_binary-chdir">chdir</h4>

(*String*): Working directory to run the binary or test in, relative to the workspace.
By default, Bazel always runs in the workspace root.
Due to implementation details, this argument must be underneath this package directory.

To run in the directory containing the `nodejs_binary` / `nodejs_test`, use

    chdir = package_name()

(or if you're in a macro, use `native.package_name()`)

WARNING: this will affect other paths passed to the program, either as arguments or in configuration files,
which are workspace-relative.
You may need `../../` segments to re-relativize such paths to the new working directory.

Defaults to `""`

<h4 id="nodejs_binary-configuration_env_vars">configuration_env_vars</h4>

(*List of strings*): Pass these configuration environment variables to the resulting binary.
Chooses a subset of the configuration environment variables (taken from `ctx.var`), which also
includes anything specified via the --define flag.
Note, this can lead to different outputs produced by this rule.

Defaults to `[]`

<h4 id="nodejs_binary-data">data</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Runtime dependencies which may be loaded during execution.

Defaults to `[]`

<h4 id="nodejs_binary-default_env_vars">default_env_vars</h4>

(*List of strings*): Default environment variables that are added to `configuration_env_vars`.

This is separate from the default of `configuration_env_vars` so that a user can set `configuration_env_vars`
without losing the defaults that should be set in most cases.

The set of default  environment variables is:

- `VERBOSE_LOGS`: use by some rules & tools to turn on debug output in their logs
- `NODE_DEBUG`: used by node.js itself to print more logs
- `RUNFILES_LIB_DEBUG`: print diagnostic message from Bazel runfiles.bash helper

Defaults to `["VERBOSE_LOGS", "NODE_DEBUG", "RUNFILES_LIB_DEBUG"]`

<h4 id="nodejs_binary-entry_point">entry_point</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>, mandatory*): The script which should be executed first, usually containing a main function.

If the entry JavaScript file belongs to the same package (as the BUILD file),
you can simply reference it by its relative name to the package directory:

```python
nodejs_binary(
    name = "my_binary",
    ...
    entry_point = ":file.js",
)
```

You can specify the entry point as a typescript file so long as you also include
the ts_library target in data:

```python
ts_library(
    name = "main",
    srcs = ["main.ts"],
)

nodejs_binary(
    name = "bin",
    data = [":main"]
    entry_point = ":main.ts",
)
```

The rule will use the corresponding `.js` output of the ts_library rule as the entry point.

If the entry point target is a rule, it should produce a single JavaScript entry file that will be passed to the nodejs_binary rule.
For example:

```python
filegroup(
    name = "entry_file",
    srcs = ["main.js"],
)

nodejs_binary(
    name = "my_binary",
    entry_point = ":entry_file",
)
```

The entry_point can also be a label in another workspace:

```python
nodejs_binary(
    name = "history-server",
    entry_point = "@npm//:node_modules/history-server/modules/cli.js",
    data = ["@npm//history-server"],
)
```


<h4 id="nodejs_binary-env">env</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Specifies additional environment variables to set when the target is executed, subject to location
expansion.

Defaults to `{}`

<h4 id="nodejs_binary-link_workspace_root">link_workspace_root</h4>

(*Boolean*): Link the workspace root to the bin_dir to support absolute requires like 'my_wksp/path/to/file'.
If source files need to be required then they can be copied to the bin_dir with copy_to_bin.

Defaults to `False`

<h4 id="nodejs_binary-templated_args">templated_args</h4>

(*List of strings*): Arguments which are passed to every execution of the program.
        To pass a node startup option, prepend it with `--node_options=`, e.g.
        `--node_options=--preserve-symlinks`.

Subject to 'Make variable' substitution. See https://docs.bazel.build/versions/master/be/make-variables.html.

1. Subject to predefined source/output path variables substitutions.

The predefined variables `execpath`, `execpaths`, `rootpath`, `rootpaths`, `location`, and `locations` take
label parameters (e.g. `$(execpath //foo:bar)`) and substitute the file paths denoted by that label.

See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_label_variables for more info.

NB: This $(location) substition returns the manifest file path which differs from the *_binary & *_test
args and genrule bazel substitions. This will be fixed in a future major release.
See docs string of `expand_location_into_runfiles` macro in `internal/common/expand_into_runfiles.bzl`
for more info.

The recommended approach is to now use `$(rootpath)` where you previously used $(location).

To get from a `$(rootpath)` to the absolute path that `$$(rlocation $(location))` returned you can either use
`$$(rlocation $(rootpath))` if you are in the `templated_args` of a `nodejs_binary` or `nodejs_test`:

BUILD.bazel:
```python
nodejs_test(
    name = "my_test",
    data = [":bootstrap.js"],
    templated_args = ["--node_options=--require=$$(rlocation $(rootpath :bootstrap.js))"],
)
```

or if you're in the context of a .js script you can pass the $(rootpath) as an argument to the script
and use the javascript runfiles helper to resolve to the absolute path:

BUILD.bazel:
```python
nodejs_test(
    name = "my_test",
    data = [":some_file"],
    entry_point = ":my_test.js",
    templated_args = ["$(rootpath :some_file)"],
)
```

my_test.js
```python
const runfiles = require(process.env['BAZEL_NODE_RUNFILES_HELPER']);
const args = process.argv.slice(2);
const some_file = runfiles.resolveWorkspaceRelative(args[0]);
```

NB: Bazel will error if it sees the single dollar sign $(rlocation path) in `templated_args` as it will try to
expand `$(rlocation)` since we now expand predefined & custom "make" variables such as `$(COMPILATION_MODE)`,
`$(BINDIR)` & `$(TARGET_CPU)` using `ctx.expand_make_variables`. See https://docs.bazel.build/versions/master/be/make-variables.html.

To prevent expansion of `$(rlocation)` write it as `$$(rlocation)`. Bazel understands `$$` to be
the string literal `$` and the expansion results in `$(rlocation)` being passed as an arg instead
of being expanded. `$(rlocation)` is then evaluated by the bash node launcher script and it calls
the `rlocation` function in the runfiles.bash helper. For example, the templated arg
`$$(rlocation $(rootpath //:some_file))` is expanded by Bazel to `$(rlocation ./some_file)` which
is then converted in bash to the absolute path of `//:some_file` in runfiles by the runfiles.bash helper
before being passed as an argument to the program.

NB: nodejs_binary and nodejs_test will preserve the legacy behavior of `$(rlocation)` so users don't
need to update to `$$(rlocation)`. This may be changed in the future.

2. Subject to predefined variables & custom variable substitutions.

Predefined "Make" variables such as $(COMPILATION_MODE) and $(TARGET_CPU) are expanded.
See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_variables.

Custom variables are also expanded including variables set through the Bazel CLI with --define=SOME_VAR=SOME_VALUE.
See https://docs.bazel.build/versions/master/be/make-variables.html#custom_variables.

Predefined genrule variables are not supported in this context.

Defaults to `[]`


## nodejs_test

**USAGE**

<pre>
nodejs_test(<a href="#nodejs_test-name">name</a>, <a href="#nodejs_test-chdir">chdir</a>, <a href="#nodejs_test-configuration_env_vars">configuration_env_vars</a>, <a href="#nodejs_test-data">data</a>, <a href="#nodejs_test-default_env_vars">default_env_vars</a>, <a href="#nodejs_test-entry_point">entry_point</a>, <a href="#nodejs_test-env">env</a>,
            <a href="#nodejs_test-expected_exit_code">expected_exit_code</a>, <a href="#nodejs_test-link_workspace_root">link_workspace_root</a>, <a href="#nodejs_test-templated_args">templated_args</a>)
</pre>


Identical to `nodejs_binary`, except this can be used with `bazel test` as well.
When the binary returns zero exit code, the test passes; otherwise it fails.

`nodejs_test` is a convenient way to write a novel kind of test based on running
your own test runner. For example, the `ts-api-guardian` library has a way to
assert the public API of a TypeScript program, and uses `nodejs_test` here:
https://github.com/angular/angular/blob/master/tools/ts-api-guardian/index.bzl

If you just want to run a standard test using a test runner from npm, use the generated
*_test target created by npm_install/yarn_install, such as `mocha_test`.
Some test runners like Karma and Jasmine have custom rules with added features, e.g. `jasmine_node_test`.

By default, Bazel runs tests with a working directory set to your workspace root.
Use the `chdir` attribute to change the working directory before the program starts.

To debug a Node.js test, we recommend saving a group of flags together in a "config".
Put this in your `tools/bazel.rc` so it's shared with your team:
```python
# Enable debugging tests with --config=debug
test:debug --test_arg=--node_options=--inspect-brk --test_output=streamed --test_strategy=exclusive --test_timeout=9999 --nocache_test_results
```

Now you can add `--config=debug` to any `bazel test` command line.
The runtime will pause before executing the program, allowing you to connect a
remote debugger.


**ATTRIBUTES**


<h4 id="nodejs_test-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this target.


<h4 id="nodejs_test-chdir">chdir</h4>

(*String*): Working directory to run the binary or test in, relative to the workspace.
By default, Bazel always runs in the workspace root.
Due to implementation details, this argument must be underneath this package directory.

To run in the directory containing the `nodejs_binary` / `nodejs_test`, use

    chdir = package_name()

(or if you're in a macro, use `native.package_name()`)

WARNING: this will affect other paths passed to the program, either as arguments or in configuration files,
which are workspace-relative.
You may need `../../` segments to re-relativize such paths to the new working directory.

Defaults to `""`

<h4 id="nodejs_test-configuration_env_vars">configuration_env_vars</h4>

(*List of strings*): Pass these configuration environment variables to the resulting binary.
Chooses a subset of the configuration environment variables (taken from `ctx.var`), which also
includes anything specified via the --define flag.
Note, this can lead to different outputs produced by this rule.

Defaults to `[]`

<h4 id="nodejs_test-data">data</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Runtime dependencies which may be loaded during execution.

Defaults to `[]`

<h4 id="nodejs_test-default_env_vars">default_env_vars</h4>

(*List of strings*): Default environment variables that are added to `configuration_env_vars`.

This is separate from the default of `configuration_env_vars` so that a user can set `configuration_env_vars`
without losing the defaults that should be set in most cases.

The set of default  environment variables is:

- `VERBOSE_LOGS`: use by some rules & tools to turn on debug output in their logs
- `NODE_DEBUG`: used by node.js itself to print more logs
- `RUNFILES_LIB_DEBUG`: print diagnostic message from Bazel runfiles.bash helper

Defaults to `["VERBOSE_LOGS", "NODE_DEBUG", "RUNFILES_LIB_DEBUG"]`

<h4 id="nodejs_test-entry_point">entry_point</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>, mandatory*): The script which should be executed first, usually containing a main function.

If the entry JavaScript file belongs to the same package (as the BUILD file),
you can simply reference it by its relative name to the package directory:

```python
nodejs_binary(
    name = "my_binary",
    ...
    entry_point = ":file.js",
)
```

You can specify the entry point as a typescript file so long as you also include
the ts_library target in data:

```python
ts_library(
    name = "main",
    srcs = ["main.ts"],
)

nodejs_binary(
    name = "bin",
    data = [":main"]
    entry_point = ":main.ts",
)
```

The rule will use the corresponding `.js` output of the ts_library rule as the entry point.

If the entry point target is a rule, it should produce a single JavaScript entry file that will be passed to the nodejs_binary rule.
For example:

```python
filegroup(
    name = "entry_file",
    srcs = ["main.js"],
)

nodejs_binary(
    name = "my_binary",
    entry_point = ":entry_file",
)
```

The entry_point can also be a label in another workspace:

```python
nodejs_binary(
    name = "history-server",
    entry_point = "@npm//:node_modules/history-server/modules/cli.js",
    data = ["@npm//history-server"],
)
```


<h4 id="nodejs_test-env">env</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Specifies additional environment variables to set when the target is executed, subject to location
expansion.

Defaults to `{}`

<h4 id="nodejs_test-expected_exit_code">expected_exit_code</h4>

(*Integer*): The expected exit code for the test. Defaults to 0.

Defaults to `0`

<h4 id="nodejs_test-link_workspace_root">link_workspace_root</h4>

(*Boolean*): Link the workspace root to the bin_dir to support absolute requires like 'my_wksp/path/to/file'.
If source files need to be required then they can be copied to the bin_dir with copy_to_bin.

Defaults to `False`

<h4 id="nodejs_test-templated_args">templated_args</h4>

(*List of strings*): Arguments which are passed to every execution of the program.
        To pass a node startup option, prepend it with `--node_options=`, e.g.
        `--node_options=--preserve-symlinks`.

Subject to 'Make variable' substitution. See https://docs.bazel.build/versions/master/be/make-variables.html.

1. Subject to predefined source/output path variables substitutions.

The predefined variables `execpath`, `execpaths`, `rootpath`, `rootpaths`, `location`, and `locations` take
label parameters (e.g. `$(execpath //foo:bar)`) and substitute the file paths denoted by that label.

See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_label_variables for more info.

NB: This $(location) substition returns the manifest file path which differs from the *_binary & *_test
args and genrule bazel substitions. This will be fixed in a future major release.
See docs string of `expand_location_into_runfiles` macro in `internal/common/expand_into_runfiles.bzl`
for more info.

The recommended approach is to now use `$(rootpath)` where you previously used $(location).

To get from a `$(rootpath)` to the absolute path that `$$(rlocation $(location))` returned you can either use
`$$(rlocation $(rootpath))` if you are in the `templated_args` of a `nodejs_binary` or `nodejs_test`:

BUILD.bazel:
```python
nodejs_test(
    name = "my_test",
    data = [":bootstrap.js"],
    templated_args = ["--node_options=--require=$$(rlocation $(rootpath :bootstrap.js))"],
)
```

or if you're in the context of a .js script you can pass the $(rootpath) as an argument to the script
and use the javascript runfiles helper to resolve to the absolute path:

BUILD.bazel:
```python
nodejs_test(
    name = "my_test",
    data = [":some_file"],
    entry_point = ":my_test.js",
    templated_args = ["$(rootpath :some_file)"],
)
```

my_test.js
```python
const runfiles = require(process.env['BAZEL_NODE_RUNFILES_HELPER']);
const args = process.argv.slice(2);
const some_file = runfiles.resolveWorkspaceRelative(args[0]);
```

NB: Bazel will error if it sees the single dollar sign $(rlocation path) in `templated_args` as it will try to
expand `$(rlocation)` since we now expand predefined & custom "make" variables such as `$(COMPILATION_MODE)`,
`$(BINDIR)` & `$(TARGET_CPU)` using `ctx.expand_make_variables`. See https://docs.bazel.build/versions/master/be/make-variables.html.

To prevent expansion of `$(rlocation)` write it as `$$(rlocation)`. Bazel understands `$$` to be
the string literal `$` and the expansion results in `$(rlocation)` being passed as an arg instead
of being expanded. `$(rlocation)` is then evaluated by the bash node launcher script and it calls
the `rlocation` function in the runfiles.bash helper. For example, the templated arg
`$$(rlocation $(rootpath //:some_file))` is expanded by Bazel to `$(rlocation ./some_file)` which
is then converted in bash to the absolute path of `//:some_file` in runfiles by the runfiles.bash helper
before being passed as an argument to the program.

NB: nodejs_binary and nodejs_test will preserve the legacy behavior of `$(rlocation)` so users don't
need to update to `$$(rlocation)`. This may be changed in the future.

2. Subject to predefined variables & custom variable substitutions.

Predefined "Make" variables such as $(COMPILATION_MODE) and $(TARGET_CPU) are expanded.
See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_variables.

Custom variables are also expanded including variables set through the Bazel CLI with --define=SOME_VAR=SOME_VALUE.
See https://docs.bazel.build/versions/master/be/make-variables.html#custom_variables.

Predefined genrule variables are not supported in this context.

Defaults to `[]`


## npm_install

**USAGE**

<pre>
npm_install(<a href="#npm_install-name">name</a>, <a href="#npm_install-args">args</a>, <a href="#npm_install-data">data</a>, <a href="#npm_install-environment">environment</a>, <a href="#npm_install-generate_local_modules_build_files">generate_local_modules_build_files</a>, <a href="#npm_install-included_files">included_files</a>,
            <a href="#npm_install-links">links</a>, <a href="#npm_install-manual_build_file_contents">manual_build_file_contents</a>, <a href="#npm_install-npm_command">npm_command</a>, <a href="#npm_install-package_json">package_json</a>, <a href="#npm_install-package_lock_json">package_lock_json</a>,
            <a href="#npm_install-package_path">package_path</a>, <a href="#npm_install-patch_args">patch_args</a>, <a href="#npm_install-patch_tool">patch_tool</a>, <a href="#npm_install-post_install_patches">post_install_patches</a>, <a href="#npm_install-pre_install_patches">pre_install_patches</a>, <a href="#npm_install-quiet">quiet</a>,
            <a href="#npm_install-repo_mapping">repo_mapping</a>, <a href="#npm_install-strict_visibility">strict_visibility</a>, <a href="#npm_install-symlink_node_modules">symlink_node_modules</a>, <a href="#npm_install-timeout">timeout</a>)
</pre>

Runs npm install during workspace setup.

This rule will set the environment variable `BAZEL_NPM_INSTALL` to '1' (unless it
set to another value in the environment attribute). Scripts may use to this to
check if yarn is being run by the `npm_install` repository rule.

**ATTRIBUTES**


<h4 id="npm_install-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this repository.


<h4 id="npm_install-args">args</h4>

(*List of strings*): Arguments passed to npm install.

See npm CLI docs https://docs.npmjs.com/cli/install.html for complete list of supported arguments.

Defaults to `[]`

<h4 id="npm_install-data">data</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Data files required by this rule.

If symlink_node_modules is True, this attribute is optional since the package manager
will run in your workspace folder. It is recommended, however, that all files that the
package manager depends on, such as `.rc` files or files used in `postinstall`, are added
symlink_node_modules is True so that the repository rule is rerun when any of these files
change.

If symlink_node_modules is False, the package manager is run in the bazel external
repository so all files that the package manager depends on must be listed.

Defaults to `[]`

<h4 id="npm_install-environment">environment</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Environment variables to set before calling the package manager.

Defaults to `{}`

<h4 id="npm_install-generate_local_modules_build_files">generate_local_modules_build_files</h4>

(*Boolean*): Enables the BUILD files auto generation for local modules installed with `file:` (npm) or `link:` (yarn)

When using a monorepo it's common to have modules that we want to use locally and
publish to an external package repository. This can be achieved using a `js_library` rule
with a `package_name` attribute defined inside the local package `BUILD` file. However,
if the project relies on the local package dependency with `file:` (npm) or `link:` (yarn) to be used outside Bazel, this
could introduce a race condition with both `npm_install` or `yarn_install` rules.

In order to overcome it, a link could be created to the package `BUILD` file from the
npm external Bazel repository (so we can use a local BUILD file instead of an auto generated one),
which require us to set `generate_local_modules_build_files = False` and complete a last step which is writing the
expected targets on that same `BUILD` file to be later used both by `npm_install` or `yarn_install`
rules, which are: `<package_name__files>`, `<package_name__nested_node_modules>`,
`<package_name__contents>`, `<package_name__typings>` and the last one just `<package_name>`. If you doubt what those targets
should look like, check the generated `BUILD` file for a given node module.

When true, the rule will follow the default behaviour of auto generating BUILD files for each `node_module` at install time.

When False, the rule will not auto generate BUILD files for `node_modules` that are installed as symlinks for local modules.

Defaults to `True`

<h4 id="npm_install-included_files">included_files</h4>

(*List of strings*): List of file extensions to be included in the npm package targets.

For example, [".js", ".d.ts", ".proto", ".json", ""].

This option is useful to limit the number of files that are inputs
to actions that depend on npm package targets. See
https://github.com/bazelbuild/bazel/issues/5153.

If set to an empty list then all files are included in the package targets.
If set to a list of extensions, only files with matching extensions are
included in the package targets. An empty string in the list is a special
string that denotes that files with no extensions such as `README` should
be included in the package targets.

This attribute applies to both the coarse `@wksp//:node_modules` target
as well as the fine grained targets such as `@wksp//foo`.

Defaults to `[]`

<h4 id="npm_install-links">links</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Targets to link as npm packages.

A mapping of npm package names to bazel targets to linked into node_modules.

If `package_path` is also set, the bazel target will be linked to the node_modules at `package_path`
along with other 3rd party npm packages from this rule.

For example,

```
yarn_install(
    name = "npm",
    package_json = "//web:package.json",
    yarn_lock = "//web:yarn.lock",
    package_path = "web",
    links = {
        "@scope/target": "//some/scoped/target",
        "target": "//some/target",
    },
)
```

creates targets in the @npm external workspace that can be used by other rules which
are linked into `web/node_modules` along side the 3rd party deps since the `project_path` is `web`.

The above links will create the targets,

```
@npm//@scope/target
@npm//target
```

that can be referenced as `data` or `deps` by other rules such as `nodejs_binary` and `ts_project`
and can be required as `@scope/target` and `target` with standard node_modules resolution at runtime,

```
nodejs_binary(
    name = "bin",
    entry_point = "bin.js",
    deps = [
        "@npm//@scope/target",
        "@npm//target"
        "@npm//other/dep"
    ],
)

ts_project(
    name = "test",
    srcs = [...],
    deps = [
        "@npm//@scope/target",
        "@npm//target"
        "@npm//other/dep"
    ],
)
```

Defaults to `{}`

<h4 id="npm_install-manual_build_file_contents">manual_build_file_contents</h4>

(*String*): Experimental attribute that can be used to override the generated BUILD.bazel file and set its contents manually.

Can be used to work-around a bazel performance issue if the
default `@wksp//:node_modules` target has too many files in it.
See https://github.com/bazelbuild/bazel/issues/5153. If
you are running into performance issues due to a large
node_modules target it is recommended to switch to using
fine grained npm dependencies.

Defaults to `""`

<h4 id="npm_install-npm_command">npm_command</h4>

(*String*): The npm command to run, to install dependencies.

            See npm docs <https://docs.npmjs.com/cli/v6/commands>

            In particular, for "ci" it says:
            > If dependencies in the package lock do not match those in package.json, npm ci will exit with an error, instead of updating the package lock.

Defaults to `"ci"`

<h4 id="npm_install-package_json">package_json</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>, mandatory*)


<h4 id="npm_install-package_lock_json">package_lock_json</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>, mandatory*)


<h4 id="npm_install-package_path">package_path</h4>

(*String*): If set, link the 3rd party node_modules dependencies under the package path specified.

In most cases, this should be the directory of the package.json file so that the linker links the node_modules
in the same location they are found in the source tree. In a future release, this will default to the package.json
directory. This is planned for 4.0: https://github.com/bazelbuild/rules_nodejs/issues/2451

Defaults to `""`

<h4 id="npm_install-patch_args">patch_args</h4>

(*List of strings*): The arguments given to the patch tool. Defaults to -p0, however -p1 will usually be needed for patches generated by git. If multiple -p arguments are specified, the last one will take effect.If arguments other than -p are specified, Bazel will fall back to use patch command line tool instead of the Bazel-native patch implementation. When falling back to patch command line tool and patch_tool attribute is not specified, `patch` will be used.

Defaults to `["-p0"]`

<h4 id="npm_install-patch_tool">patch_tool</h4>

(*String*): The patch(1) utility to use. If this is specified, Bazel will use the specifed patch tool instead of the Bazel-native patch implementation.

Defaults to `""`

<h4 id="npm_install-post_install_patches">post_install_patches</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Patch files to apply after running package manager.

This can be used to make changes to installed packages after the package manager runs.

File paths in patches should be relative to workspace root.

Defaults to `[]`

<h4 id="npm_install-pre_install_patches">pre_install_patches</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Patch files to apply before running package manager.

This can be used to make changes to package.json or other data files passed in before running the
package manager.

File paths in patches should be relative to workspace root.

Defaults to `[]`

<h4 id="npm_install-quiet">quiet</h4>

(*Boolean*): If stdout and stderr should be printed to the terminal.

Defaults to `True`

<h4 id="npm_install-repo_mapping">repo_mapping</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>, mandatory*): A dictionary from local repository name to global repository name. This allows controls over workspace dependency resolution for dependencies of this repository.<p>For example, an entry `"@foo": "@bar"` declares that, for any time this repository depends on `@foo` (such as a dependency on `@foo//some:target`, it should actually resolve that dependency within globally-declared `@bar` (`@bar//some:target`).


<h4 id="npm_install-strict_visibility">strict_visibility</h4>

(*Boolean*): Turn on stricter visibility for generated BUILD.bazel files

When enabled, only dependencies within the given `package.json` file are given public visibility.
All transitive dependencies are given limited visibility, enforcing that all direct dependencies are
listed in the `package.json` file.

Defaults to `True`

<h4 id="npm_install-symlink_node_modules">symlink_node_modules</h4>

(*Boolean*): Turn symlinking of node_modules on

This requires the use of Bazel 0.26.0 and the experimental
managed_directories feature.

When true, the package manager will run in the package.json folder
and the resulting node_modules folder will be symlinked into the
external repository create by this rule.

When false, the package manager will run in the external repository
created by this rule and any files other than the package.json file and
the lock file that are required for it to run should be listed in the
data attribute.

Defaults to `True`

<h4 id="npm_install-timeout">timeout</h4>

(*Integer*): Maximum duration of the package manager execution in seconds.

Defaults to `3600`


## pkg_npm

**USAGE**

<pre>
pkg_npm(<a href="#pkg_npm-name">name</a>, <a href="#pkg_npm-deps">deps</a>, <a href="#pkg_npm-nested_packages">nested_packages</a>, <a href="#pkg_npm-node_context_data">node_context_data</a>, <a href="#pkg_npm-package_name">package_name</a>, <a href="#pkg_npm-package_path">package_path</a>, <a href="#pkg_npm-srcs">srcs</a>,
        <a href="#pkg_npm-substitutions">substitutions</a>, <a href="#pkg_npm-tgz">tgz</a>, <a href="#pkg_npm-vendor_external">vendor_external</a>)
</pre>

The pkg_npm rule creates a directory containing a publishable npm artifact.

Example:

```python
load("@build_bazel_rules_nodejs//:index.bzl", "pkg_npm")

pkg_npm(
    name = "my_package",
    srcs = ["package.json"],
    deps = [":my_typescript_lib"],
    substitutions = {"//internal/": "//"},
)
```

You can use a pair of `// BEGIN-INTERNAL ... // END-INTERNAL` comments to mark regions of files that should be elided during publishing.
For example:

```javascript
function doThing() {
    // BEGIN-INTERNAL
    // This is a secret internal-only comment
    doInternalOnlyThing();
    // END-INTERNAL
}
```

With the Bazel stamping feature, pkg_npm will replace any placeholder version in your package with the actual version control tag.
See the [stamping documentation](https://github.com/bazelbuild/rules_nodejs/blob/master/docs/index.md#stamping)

Usage:

`pkg_npm` yields four labels. Build the package directory using the default label:

```sh
$ bazel build :my_package
Target //:my_package up-to-date:
  bazel-out/fastbuild/bin/my_package
$ ls -R bazel-out/fastbuild/bin/my_package
```

Dry-run of publishing to npm, calling `npm pack` (it builds the package first if needed):

```sh
$ bazel run :my_package.pack
INFO: Running command line: bazel-out/fastbuild/bin/my_package.pack
my-package-name-1.2.3.tgz
$ tar -tzf my-package-name-1.2.3.tgz
```

Actually publish the package with `npm publish` (also builds first):

```sh
# Check login credentials
$ bazel run @nodejs//:npm_node_repositories who
# Publishes the package
$ bazel run :my_package.publish
```

You can pass arguments to npm by escaping them from Bazel using a double-hyphen, for example:

`bazel run my_package.publish -- --tag=next`

It is also possible to use the resulting tar file file from the `.pack` as an action input via the `.tar` label.
To make use of this label, the `tgz` attribute must be set, and the generating `pkg_npm` rule must have a valid `package.json` file
as part of its sources:

```python
pkg_npm(
    name = "my_package",
    srcs = ["package.json"],
    deps = [":my_typescript_lib"],
    tgz = "my_package.tgz",
)

my_rule(
    name = "foo",
    srcs = [
        "//:my_package.tar",
    ],
)
```


**ATTRIBUTES**


<h4 id="pkg_npm-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this target.


<h4 id="pkg_npm-deps">deps</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Other targets which produce files that should be included in the package, such as `rollup_bundle`

Defaults to `[]`

<h4 id="pkg_npm-nested_packages">nested_packages</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Other pkg_npm rules whose content is copied into this package.

Defaults to `[]`

<h4 id="pkg_npm-node_context_data">node_context_data</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>*): Provides info about the build context, such as stamping.
        
By default it reads from the bazel command line, such as the `--stamp` argument.
Use this to override values for this target, such as enabling or disabling stamping.
You can use the `node_context_data` rule in `@build_bazel_rules_nodejs//internal/node:context.bzl`
to create a NodeContextInfo.  The dependencies of this attribute must provide: NodeContextInfo


Defaults to `@build_bazel_rules_nodejs//internal:node_context_data`

<h4 id="pkg_npm-package_name">package_name</h4>

(*String*): Optional package_name that this npm package may be imported as.

Defaults to `""`

<h4 id="pkg_npm-package_path">package_path</h4>

(*String*): The directory in the workspace to link to.
If set, link this pkg_npm to the node_modules under the package path specified.
If unset, the default is to link to the node_modules root of the workspace.

Defaults to `""`

<h4 id="pkg_npm-srcs">srcs</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Files inside this directory which are simply copied into the package.

Defaults to `[]`

<h4 id="pkg_npm-substitutions">substitutions</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Key-value pairs which are replaced in all the files while building the package.
        
You can use values from the workspace status command using curly braces, for example
`{"0.0.0-PLACEHOLDER": "{STABLE_GIT_VERSION}"}`.

See the section on stamping in the [README](stamping)

Defaults to `{}`

<h4 id="pkg_npm-tgz">tgz</h4>

(*String*): If set, will create a `.tgz` file that can be used as an input to another rule, the tar will be given the name assigned to this attribute.

        NOTE: If this attribute is set, a valid `package.json` file must be included in the sources of this target

Defaults to `""`

<h4 id="pkg_npm-vendor_external">vendor_external</h4>

(*List of strings*): External workspaces whose contents should be vendored into this workspace.
        Avoids `external/foo` path segments in the resulting package.

Defaults to `[]`


## pkg_web

**USAGE**

<pre>
pkg_web(<a href="#pkg_web-name">name</a>, <a href="#pkg_web-additional_root_paths">additional_root_paths</a>, <a href="#pkg_web-node_context_data">node_context_data</a>, <a href="#pkg_web-srcs">srcs</a>, <a href="#pkg_web-substitutions">substitutions</a>)
</pre>

Assembles a web application from source files.

**ATTRIBUTES**


<h4 id="pkg_web-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this target.


<h4 id="pkg_web-additional_root_paths">additional_root_paths</h4>

(*List of strings*): Path prefixes to strip off all srcs relative to the root of the repo, in addition to the current package. Longest wins.

Defaults to `[]`

<h4 id="pkg_web-node_context_data">node_context_data</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>*): Provides info about the build context, such as stamping.
        
By default it reads from the bazel command line, such as the `--stamp` argument.
Use this to override values for this target, such as enabling or disabling stamping.
You can use the `node_context_data` rule in `@build_bazel_rules_nodejs//internal/node:context.bzl`
to create a NodeContextInfo.  The dependencies of this attribute must provide: NodeContextInfo


Defaults to `@build_bazel_rules_nodejs//internal:node_context_data`

<h4 id="pkg_web-srcs">srcs</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Files which should be copied into the package

Defaults to `[]`

<h4 id="pkg_web-substitutions">substitutions</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Key-value pairs which are replaced in all the files while building the package.

You can use values from the workspace status command using curly braces, for example
`{"0.0.0-PLACEHOLDER": "{STABLE_GIT_VERSION}"}`.
See the section on stamping in the README.

Defaults to `{}`


## yarn_install

**USAGE**

<pre>
yarn_install(<a href="#yarn_install-name">name</a>, <a href="#yarn_install-args">args</a>, <a href="#yarn_install-data">data</a>, <a href="#yarn_install-environment">environment</a>, <a href="#yarn_install-frozen_lockfile">frozen_lockfile</a>, <a href="#yarn_install-generate_local_modules_build_files">generate_local_modules_build_files</a>,
             <a href="#yarn_install-included_files">included_files</a>, <a href="#yarn_install-links">links</a>, <a href="#yarn_install-manual_build_file_contents">manual_build_file_contents</a>, <a href="#yarn_install-package_json">package_json</a>, <a href="#yarn_install-package_path">package_path</a>,
             <a href="#yarn_install-patch_args">patch_args</a>, <a href="#yarn_install-patch_tool">patch_tool</a>, <a href="#yarn_install-post_install_patches">post_install_patches</a>, <a href="#yarn_install-pre_install_patches">pre_install_patches</a>, <a href="#yarn_install-quiet">quiet</a>, <a href="#yarn_install-repo_mapping">repo_mapping</a>,
             <a href="#yarn_install-strict_visibility">strict_visibility</a>, <a href="#yarn_install-symlink_node_modules">symlink_node_modules</a>, <a href="#yarn_install-timeout">timeout</a>, <a href="#yarn_install-use_global_yarn_cache">use_global_yarn_cache</a>, <a href="#yarn_install-yarn_lock">yarn_lock</a>)
</pre>

Runs yarn install during workspace setup.

This rule will set the environment variable `BAZEL_YARN_INSTALL` to '1' (unless it
set to another value in the environment attribute). Scripts may use to this to
check if yarn is being run by the `yarn_install` repository rule.

**ATTRIBUTES**


<h4 id="yarn_install-name">name</h4>

(*<a href="https://bazel.build/docs/build-ref.html#name">Name</a>, mandatory*): A unique name for this repository.


<h4 id="yarn_install-args">args</h4>

(*List of strings*): Arguments passed to yarn install.

See yarn CLI docs https://yarnpkg.com/en/docs/cli/install for complete list of supported arguments.

Defaults to `[]`

<h4 id="yarn_install-data">data</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Data files required by this rule.

If symlink_node_modules is True, this attribute is optional since the package manager
will run in your workspace folder. It is recommended, however, that all files that the
package manager depends on, such as `.rc` files or files used in `postinstall`, are added
symlink_node_modules is True so that the repository rule is rerun when any of these files
change.

If symlink_node_modules is False, the package manager is run in the bazel external
repository so all files that the package manager depends on must be listed.

Defaults to `[]`

<h4 id="yarn_install-environment">environment</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Environment variables to set before calling the package manager.

Defaults to `{}`

<h4 id="yarn_install-frozen_lockfile">frozen_lockfile</h4>

(*Boolean*): Use the `--frozen-lockfile` flag for yarn.

Don't generate a `yarn.lock` lockfile and fail if an update is needed.

This flag enables an exact install of the version that is specified in the `yarn.lock`
file. This helps to have reproducible builds across builds.

To update a dependency or install a new one run the `yarn install` command with the
vendored yarn binary. `bazel run @nodejs//:yarn install`. You can pass the options like
`bazel run @nodejs//:yarn install -- -D <dep-name>`.

Defaults to `True`

<h4 id="yarn_install-generate_local_modules_build_files">generate_local_modules_build_files</h4>

(*Boolean*): Enables the BUILD files auto generation for local modules installed with `file:` (npm) or `link:` (yarn)

When using a monorepo it's common to have modules that we want to use locally and
publish to an external package repository. This can be achieved using a `js_library` rule
with a `package_name` attribute defined inside the local package `BUILD` file. However,
if the project relies on the local package dependency with `file:` (npm) or `link:` (yarn) to be used outside Bazel, this
could introduce a race condition with both `npm_install` or `yarn_install` rules.

In order to overcome it, a link could be created to the package `BUILD` file from the
npm external Bazel repository (so we can use a local BUILD file instead of an auto generated one),
which require us to set `generate_local_modules_build_files = False` and complete a last step which is writing the
expected targets on that same `BUILD` file to be later used both by `npm_install` or `yarn_install`
rules, which are: `<package_name__files>`, `<package_name__nested_node_modules>`,
`<package_name__contents>`, `<package_name__typings>` and the last one just `<package_name>`. If you doubt what those targets
should look like, check the generated `BUILD` file for a given node module.

When true, the rule will follow the default behaviour of auto generating BUILD files for each `node_module` at install time.

When False, the rule will not auto generate BUILD files for `node_modules` that are installed as symlinks for local modules.

Defaults to `True`

<h4 id="yarn_install-included_files">included_files</h4>

(*List of strings*): List of file extensions to be included in the npm package targets.

For example, [".js", ".d.ts", ".proto", ".json", ""].

This option is useful to limit the number of files that are inputs
to actions that depend on npm package targets. See
https://github.com/bazelbuild/bazel/issues/5153.

If set to an empty list then all files are included in the package targets.
If set to a list of extensions, only files with matching extensions are
included in the package targets. An empty string in the list is a special
string that denotes that files with no extensions such as `README` should
be included in the package targets.

This attribute applies to both the coarse `@wksp//:node_modules` target
as well as the fine grained targets such as `@wksp//foo`.

Defaults to `[]`

<h4 id="yarn_install-links">links</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>*): Targets to link as npm packages.

A mapping of npm package names to bazel targets to linked into node_modules.

If `package_path` is also set, the bazel target will be linked to the node_modules at `package_path`
along with other 3rd party npm packages from this rule.

For example,

```
yarn_install(
    name = "npm",
    package_json = "//web:package.json",
    yarn_lock = "//web:yarn.lock",
    package_path = "web",
    links = {
        "@scope/target": "//some/scoped/target",
        "target": "//some/target",
    },
)
```

creates targets in the @npm external workspace that can be used by other rules which
are linked into `web/node_modules` along side the 3rd party deps since the `project_path` is `web`.

The above links will create the targets,

```
@npm//@scope/target
@npm//target
```

that can be referenced as `data` or `deps` by other rules such as `nodejs_binary` and `ts_project`
and can be required as `@scope/target` and `target` with standard node_modules resolution at runtime,

```
nodejs_binary(
    name = "bin",
    entry_point = "bin.js",
    deps = [
        "@npm//@scope/target",
        "@npm//target"
        "@npm//other/dep"
    ],
)

ts_project(
    name = "test",
    srcs = [...],
    deps = [
        "@npm//@scope/target",
        "@npm//target"
        "@npm//other/dep"
    ],
)
```

Defaults to `{}`

<h4 id="yarn_install-manual_build_file_contents">manual_build_file_contents</h4>

(*String*): Experimental attribute that can be used to override the generated BUILD.bazel file and set its contents manually.

Can be used to work-around a bazel performance issue if the
default `@wksp//:node_modules` target has too many files in it.
See https://github.com/bazelbuild/bazel/issues/5153. If
you are running into performance issues due to a large
node_modules target it is recommended to switch to using
fine grained npm dependencies.

Defaults to `""`

<h4 id="yarn_install-package_json">package_json</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>, mandatory*)


<h4 id="yarn_install-package_path">package_path</h4>

(*String*): If set, link the 3rd party node_modules dependencies under the package path specified.

In most cases, this should be the directory of the package.json file so that the linker links the node_modules
in the same location they are found in the source tree. In a future release, this will default to the package.json
directory. This is planned for 4.0: https://github.com/bazelbuild/rules_nodejs/issues/2451

Defaults to `""`

<h4 id="yarn_install-patch_args">patch_args</h4>

(*List of strings*): The arguments given to the patch tool. Defaults to -p0, however -p1 will usually be needed for patches generated by git. If multiple -p arguments are specified, the last one will take effect.If arguments other than -p are specified, Bazel will fall back to use patch command line tool instead of the Bazel-native patch implementation. When falling back to patch command line tool and patch_tool attribute is not specified, `patch` will be used.

Defaults to `["-p0"]`

<h4 id="yarn_install-patch_tool">patch_tool</h4>

(*String*): The patch(1) utility to use. If this is specified, Bazel will use the specifed patch tool instead of the Bazel-native patch implementation.

Defaults to `""`

<h4 id="yarn_install-post_install_patches">post_install_patches</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Patch files to apply after running package manager.

This can be used to make changes to installed packages after the package manager runs.

File paths in patches should be relative to workspace root.

Defaults to `[]`

<h4 id="yarn_install-pre_install_patches">pre_install_patches</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">List of labels</a>*): Patch files to apply before running package manager.

This can be used to make changes to package.json or other data files passed in before running the
package manager.

File paths in patches should be relative to workspace root.

Defaults to `[]`

<h4 id="yarn_install-quiet">quiet</h4>

(*Boolean*): If stdout and stderr should be printed to the terminal.

Defaults to `True`

<h4 id="yarn_install-repo_mapping">repo_mapping</h4>

(*<a href="https://bazel.build/docs/skylark/lib/dict.html">Dictionary: String -> String</a>, mandatory*): A dictionary from local repository name to global repository name. This allows controls over workspace dependency resolution for dependencies of this repository.<p>For example, an entry `"@foo": "@bar"` declares that, for any time this repository depends on `@foo` (such as a dependency on `@foo//some:target`, it should actually resolve that dependency within globally-declared `@bar` (`@bar//some:target`).


<h4 id="yarn_install-strict_visibility">strict_visibility</h4>

(*Boolean*): Turn on stricter visibility for generated BUILD.bazel files

When enabled, only dependencies within the given `package.json` file are given public visibility.
All transitive dependencies are given limited visibility, enforcing that all direct dependencies are
listed in the `package.json` file.

Defaults to `True`

<h4 id="yarn_install-symlink_node_modules">symlink_node_modules</h4>

(*Boolean*): Turn symlinking of node_modules on

This requires the use of Bazel 0.26.0 and the experimental
managed_directories feature.

When true, the package manager will run in the package.json folder
and the resulting node_modules folder will be symlinked into the
external repository create by this rule.

When false, the package manager will run in the external repository
created by this rule and any files other than the package.json file and
the lock file that are required for it to run should be listed in the
data attribute.

Defaults to `True`

<h4 id="yarn_install-timeout">timeout</h4>

(*Integer*): Maximum duration of the package manager execution in seconds.

Defaults to `3600`

<h4 id="yarn_install-use_global_yarn_cache">use_global_yarn_cache</h4>

(*Boolean*): Use the global yarn cache on the system.

The cache lets you avoid downloading packages multiple times.
However, it can introduce non-hermeticity, and the yarn cache can
have bugs.

Disabling this attribute causes every run of yarn to have a unique
cache_directory.

If True, this rule will pass `--mutex network` to yarn to ensure that
the global cache can be shared by parallelized yarn_install rules.

If False, this rule will pass `--cache-folder /path/to/external/repository/__yarn_cache`
to yarn so that the local cache is contained within the external repository.

Defaults to `True`

<h4 id="yarn_install-yarn_lock">yarn_lock</h4>

(*<a href="https://bazel.build/docs/build-ref.html#labels">Label</a>, mandatory*)



## check_bazel_version

**USAGE**

<pre>
check_bazel_version(<a href="#check_bazel_version-minimum_bazel_version">minimum_bazel_version</a>, <a href="#check_bazel_version-message">message</a>)
</pre>

    Verify the users Bazel version is at least the given one.

This can be used in rule implementations that depend on changes in Bazel,
to warn users about a mismatch between the rule and their installed Bazel
version.

This should *not* be used in users WORKSPACE files. To locally pin your
Bazel version, just create the .bazelversion file in your workspace.


**PARAMETERS**


<h4 id="check_bazel_version-minimum_bazel_version">minimum_bazel_version</h4>

a string indicating the minimum version



<h4 id="check_bazel_version-message">message</h4>

optional string to print to your users, could be used to help them update

Defaults to `""`


## copy_to_bin

**USAGE**

<pre>
copy_to_bin(<a href="#copy_to_bin-name">name</a>, <a href="#copy_to_bin-srcs">srcs</a>, <a href="#copy_to_bin-kwargs">kwargs</a>)
</pre>

Copies a source file to bazel-bin at the same workspace-relative path.

e.g. `<workspace_root>/foo/bar/a.txt -> <bazel-bin>/foo/bar/a.txt`

This is useful to populate the output folder with all files needed at runtime, even
those which aren't outputs of a Bazel rule.

This way you can run a binary in the output folder (execroot or runfiles_root)
without that program needing to rely on a runfiles helper library or be aware that
files are divided between the source tree and the output tree.


**PARAMETERS**


<h4 id="copy_to_bin-name">name</h4>

Name of the rule.



<h4 id="copy_to_bin-srcs">srcs</h4>

A List of Labels. File(s) to to copy.



<h4 id="copy_to_bin-kwargs">kwargs</h4>

further keyword arguments, e.g. `visibility`




## generated_file_test

**USAGE**

<pre>
generated_file_test(<a href="#generated_file_test-name">name</a>, <a href="#generated_file_test-generated">generated</a>, <a href="#generated_file_test-src">src</a>, <a href="#generated_file_test-substring_search">substring_search</a>, <a href="#generated_file_test-src_dbg">src_dbg</a>, <a href="#generated_file_test-kwargs">kwargs</a>)
</pre>

Tests that a file generated by Bazel has identical content to a file in the workspace.

This is useful for testing, where a "snapshot" or "golden" file is checked in,
so that you can code review changes to the generated output.


**PARAMETERS**


<h4 id="generated_file_test-name">name</h4>

Name of the rule.



<h4 id="generated_file_test-generated">generated</h4>

a Label of the output file generated by another rule



<h4 id="generated_file_test-src">src</h4>

Label of the source file in the workspace



<h4 id="generated_file_test-substring_search">substring_search</h4>

When true, creates a test that will fail only if the golden file is not found
anywhere within the generated file. Note that the .update rule is not generated in substring mode.

Defaults to `False`

<h4 id="generated_file_test-src_dbg">src_dbg</h4>

if the build uses `--compilation_mode dbg` then some rules will produce different output.
In this case you can specify what the dbg version of the output should look like

Defaults to `None`

<h4 id="generated_file_test-kwargs">kwargs</h4>

extra arguments passed to the underlying nodejs_test




## js_library

**USAGE**

<pre>
js_library(<a href="#js_library-name">name</a>, <a href="#js_library-srcs">srcs</a>, <a href="#js_library-package_name">package_name</a>, <a href="#js_library-package_path">package_path</a>, <a href="#js_library-deps">deps</a>, <a href="#js_library-kwargs">kwargs</a>)
</pre>

Groups JavaScript code so that it can be depended on like an npm package.

`js_library` is intended to be used internally within Bazel, such as between two libraries in your monorepo.
This rule doesn't perform any build steps ("actions") so it is similar to a `filegroup`.
However it provides several Bazel "Providers" for interop with other rules.

> Compare this to `pkg_npm` which just produces a directory output, and therefore can't expose individual
> files to downstream targets and causes a cascading re-build of all transitive dependencies when any file
> changes. Also `pkg_npm` is intended to publish your code for external usage outside of Bazel, like
> by publishing to npm or artifactory, while `js_library` is for internal dependencies within your repo.

`js_library` also copies any source files into the bazel-out folder.
This is the same behavior as the `copy_to_bin` rule.
By copying the complete package to the output tree, we ensure that the linker (our `npm link` equivalent)
will make your source files available in the node_modules tree where resolvers expect them.
It also means you can have relative imports between the files
rather than being forced to use Bazel's "Runfiles" semantics where any program might need a helper library
to resolve files between the logical union of the source tree and the output tree.

### Example

A typical example usage of `js_library` is to expose some sources with a package name:

```python
ts_project(
    name = "compile_ts",
    srcs = glob(["*.ts"]),
)

js_library(
    name = "my_pkg",
    # Code that depends on this target can import from "@myco/mypkg"
    package_name = "@myco/mypkg",
    # Consumers might need fields like "main" or "typings"
    srcs = ["package.json"],
    # The .js and .d.ts outputs from above will be part of the package
    deps = [":compile_ts"],
)
```

> To help work with "named AMD" modules as required by `concatjs_devserver` and other Google-style "concatjs" rules,
> `js_library` has some undocumented advanced features you can find in the source code or in our examples.
> These should not be considered a public API and aren't subject to our usual support and semver guarantees.

### Outputs

Like all Bazel rules it produces a default output by providing [DefaultInfo].
You'll get these outputs if you include this in the `srcs` of a typical rule like `filegroup`,
and these will be the printed result when you `bazel build //some:js_library_target`.
The default outputs are all of:
- [DefaultInfo] produced by targets in `deps`
- A copy of all sources (InputArtifacts from your source tree) in the bazel-out directory

When there are TypeScript typings files, `js_library` provides [DeclarationInfo](#declarationinfo)
so this target can be a dependency of a TypeScript rule. This includes any `.d.ts` files in `srcs` as well
as transitive ones from `deps`.
It will also provide [OutputGroupInfo] with a "types" field, so you can select the typings outputs with
`bazel build //some:js_library_target --output_groups=types` or with a `filegroup` rule using the
[output_group] attribute.

In order to work with the linker (similar to `npm link` for first-party monorepo deps), `js_library` provides
[LinkablePackageInfo](#linkablepackageinfo) for use with our "linker" that makes this package importable.

It also provides:
- [ExternalNpmPackageInfo](#externalnpmpackageinfo) to interop with rules that expect third-party npm packages.
- [JSModuleInfo](#jsmoduleinfo) so rules like bundlers can collect the transitive set of .js files
- [JSNamedModuleInfo](#jsnamedmoduleinfo) for rules that expect named AMD or `goog.module` format JS

[OutputGroupInfo]: https://docs.bazel.build/versions/master/skylark/lib/OutputGroupInfo.html
[DefaultInfo]: https://docs.bazel.build/versions/master/skylark/lib/DefaultInfo.html
[output_group]: https://docs.bazel.build/versions/master/be/general.html#filegroup.output_group


**PARAMETERS**


<h4 id="js_library-name">name</h4>

The name for the target



<h4 id="js_library-srcs">srcs</h4>

The list of files that comprise the package

Defaults to `[]`

<h4 id="js_library-package_name">package_name</h4>

The name it will be imported by. Should match the "name" field in the package.json file.

If package_name == "$node_modules$" this indictates that this js_library target is one or more external npm
packages in node_modules. This is a special case that used be covered by the internal only
`external_npm_package` attribute. NB: '$' is an illegal character
for npm packages names so this reserved name will not conflict with any valid package_name values

This is used by the yarn_install & npm_install repository rules for npm dependencies installed by
yarn & npm. When true, js_library will provide ExternalNpmPackageInfo.

It can also be used for user-managed npm dependencies if node_modules is layed out outside of bazel.
For example,

js_library(
    name = "node_modules",
    srcs = glob(
        include = [
            "node_modules/**/*.js",
            "node_modules/**/*.d.ts",
            "node_modules/**/*.json",
            "node_modules/.bin/*",
        ],
        exclude = [
            # Files under test & docs may contain file names that
            # are not legal Bazel labels (e.g.,
            # node_modules/ecstatic/test/public/ä¸­æ/æªæ¡.html)
            "node_modules/**/test/**",
            "node_modules/**/docs/**",
            # Files with spaces in the name are not legal Bazel labels
            "node_modules/**/* */**",
            "node_modules/**/* *",
        ],
    ),
    # Special value to provide ExternalNpmPackageInfo which is used by downstream
    # rules that use these npm dependencies
    package_name = "$node_modules$",
)

See `examples/user_managed_deps` for a working example of user-managed npm dependencies.

Defaults to `None`

<h4 id="js_library-package_path">package_path</h4>

The directory in the workspace to link to.
If set, link this js_library to the node_modules under the package path specified.
If unset, the default is to link to the node_modules root of the workspace.

Defaults to `""`

<h4 id="js_library-deps">deps</h4>

Other targets that provide JavaScript code

Defaults to `[]`

<h4 id="js_library-kwargs">kwargs</h4>

Other attributes




## npm_package_bin

**USAGE**

<pre>
npm_package_bin(<a href="#npm_package_bin-tool">tool</a>, <a href="#npm_package_bin-package">package</a>, <a href="#npm_package_bin-package_bin">package_bin</a>, <a href="#npm_package_bin-data">data</a>, <a href="#npm_package_bin-env">env</a>, <a href="#npm_package_bin-outs">outs</a>, <a href="#npm_package_bin-args">args</a>, <a href="#npm_package_bin-stderr">stderr</a>, <a href="#npm_package_bin-stdout">stdout</a>, <a href="#npm_package_bin-exit_code_out">exit_code_out</a>,
                <a href="#npm_package_bin-output_dir">output_dir</a>, <a href="#npm_package_bin-link_workspace_root">link_workspace_root</a>, <a href="#npm_package_bin-chdir">chdir</a>, <a href="#npm_package_bin-kwargs">kwargs</a>)
</pre>

Run an arbitrary npm package binary (e.g. a program under node_modules/.bin/*) under Bazel.

It must produce outputs. If you just want to run a program with `bazel run`, use the nodejs_binary rule.

This is like a genrule() except that it runs our launcher script that first
links the node_modules tree before running the program.

By default, Bazel runs actions with a working directory set to your workspace root.
Use the `chdir` attribute to change the working directory before the program runs.

This is a great candidate to wrap with a macro, as documented:
https://docs.bazel.build/versions/master/skylark/macros.html#full-example


**PARAMETERS**


<h4 id="npm_package_bin-tool">tool</h4>

a label for a binary to run, like `@npm//terser/bin:terser`. This is the longer form of package/package_bin.
Note that you can also refer to a binary in your local workspace.

Defaults to `None`

<h4 id="npm_package_bin-package">package</h4>

an npm package whose binary to run, like "terser". Assumes your node_modules are installed in a workspace called "npm"

Defaults to `None`

<h4 id="npm_package_bin-package_bin">package_bin</h4>

the "bin" entry from `package` that should be run. By default package_bin is the same string as `package`

Defaults to `None`

<h4 id="npm_package_bin-data">data</h4>

similar to [genrule.srcs](https://docs.bazel.build/versions/master/be/general.html#genrule.srcs)
may also include targets that produce or reference npm packages which are needed by the tool

Defaults to `[]`

<h4 id="npm_package_bin-env">env</h4>

specifies additional environment variables to set when the target is executed

Defaults to `{}`

<h4 id="npm_package_bin-outs">outs</h4>

similar to [genrule.outs](https://docs.bazel.build/versions/master/be/general.html#genrule.outs)

Defaults to `[]`

<h4 id="npm_package_bin-args">args</h4>

Command-line arguments to the tool.

Subject to 'Make variable' substitution. See https://docs.bazel.build/versions/master/be/make-variables.html.

1. Predefined source/output path substitions is applied first:

See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_label_variables.

Use $(execpath) $(execpaths) to expand labels to the execroot (where Bazel runs build actions).

Use $(rootpath) $(rootpaths) to expand labels to the runfiles path that a built binary can use
to find its dependencies.

Since npm_package_bin is used primarily for build actions, in most cases you'll want to
use $(execpath) or $(execpaths) to expand locations.

Using $(location) and $(locations) expansions is not recommended as these are a synonyms
for either $(execpath) or $(rootpath) depending on the context.

2. "Make" variables are expanded second:

Predefined "Make" variables such as $(COMPILATION_MODE) and $(TARGET_CPU) are expanded.
See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_variables.

Like genrule, you may also use some syntax sugar for locations.

- `$@`: if you have only one output file, the location of the output
- `$(@D)`: The output directory. If output_dir=False and there is only one file name in outs, this expands to the directory
    containing that file. If there are multiple files, this instead expands to the package's root directory in the genfiles
    tree, even if all generated files belong to the same subdirectory! If output_dir=True then this corresponds
    to the output directory which is the $(RULEDIR)/{target_name}.
- `$(RULEDIR)`: the root output directory of the rule, corresponding with its package
    (can be used with output_dir=True or False)

See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_genrule_variables.

Custom variables are also expanded including variables set through the Bazel CLI with --define=SOME_VAR=SOME_VALUE.
See https://docs.bazel.build/versions/master/be/make-variables.html#custom_variables.

Defaults to `[]`

<h4 id="npm_package_bin-stderr">stderr</h4>

set to capture the stderr of the binary to a file, which can later be used as an input to another target
subject to the same semantics as `outs`

Defaults to `None`

<h4 id="npm_package_bin-stdout">stdout</h4>

set to capture the stdout of the binary to a file, which can later be used as an input to another target
subject to the same semantics as `outs`

Defaults to `None`

<h4 id="npm_package_bin-exit_code_out">exit_code_out</h4>

set to capture the exit code of the binary to a file, which can later be used as an input to another target
subject to the same semantics as `outs`. Note that setting this will force the binary to exit 0.
If the binary creates outputs and these are declared, they must still be created

Defaults to `None`

<h4 id="npm_package_bin-output_dir">output_dir</h4>

set to True if you want the output to be a directory
Exactly one of `outs`, `output_dir` may be used.
If you output a directory, there can only be one output, which will be a directory named the same as the target.

Defaults to `False`

<h4 id="npm_package_bin-link_workspace_root">link_workspace_root</h4>

Link the workspace root to the bin_dir to support absolute requires like 'my_wksp/path/to/file'.
If source files need to be required then they can be copied to the bin_dir with copy_to_bin.

Defaults to `False`

<h4 id="npm_package_bin-chdir">chdir</h4>

Working directory to run the binary or test in, relative to the workspace.

By default, Bazel always runs in the workspace root.

To run in the directory containing the `npm_package_bin` under the source tree, use
`chdir = package_name()`
(or if you're in a macro, use `native.package_name()`).

To run in the output directory where the npm_package_bin writes outputs, use
`chdir = "$(RULEDIR)"`

WARNING: this will affect other paths passed to the program, either as arguments or in configuration files,
which are workspace-relative.
You may need `../../` segments to re-relativize such paths to the new working directory.
In a `BUILD` file you could do something like this to point to the output path:

```python
_package_segments = len(package_name().split("/"))
npm_package_bin(
    ...
    chdir = package_name(),
    # ../.. segments to re-relative paths from the chdir back to workspace
    args = ["/".join([".."] * _package_segments + ["$@"])],
)
```

Defaults to `None`

<h4 id="npm_package_bin-kwargs">kwargs</h4>

additional undocumented keyword args




## params_file

**USAGE**

<pre>
params_file(<a href="#params_file-name">name</a>, <a href="#params_file-out">out</a>, <a href="#params_file-args">args</a>, <a href="#params_file-data">data</a>, <a href="#params_file-newline">newline</a>, <a href="#params_file-kwargs">kwargs</a>)
</pre>

Generates a UTF-8 encoded params file from a list of arguments.

Handles variable substitutions for args.


**PARAMETERS**


<h4 id="params_file-name">name</h4>

Name of the rule.



<h4 id="params_file-out">out</h4>

Path of the output file, relative to this package.



<h4 id="params_file-args">args</h4>

Arguments to concatenate into a params file.

Subject to 'Make variable' substitution. See https://docs.bazel.build/versions/master/be/make-variables.html.

1. Subject to predefined source/output path variables substitutions.

The predefined variables `execpath`, `execpaths`, `rootpath`, `rootpaths`, `location`, and `locations` take
label parameters (e.g. `$(execpath //foo:bar)`) and substitute the file paths denoted by that label.

See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_label_variables for more info.

NB: This $(location) substition returns the manifest file path which differs from the *_binary & *_test
args and genrule bazel substitions. This will be fixed in a future major release.
See docs string of `expand_location_into_runfiles` macro in `internal/common/expand_into_runfiles.bzl`
for more info.

2. Subject to predefined variables & custom variable substitutions.

Predefined "Make" variables such as $(COMPILATION_MODE) and $(TARGET_CPU) are expanded.
See https://docs.bazel.build/versions/master/be/make-variables.html#predefined_variables.

Custom variables are also expanded including variables set through the Bazel CLI with --define=SOME_VAR=SOME_VALUE.
See https://docs.bazel.build/versions/master/be/make-variables.html#custom_variables.

Predefined genrule variables are not supported in this context.

Defaults to `[]`

<h4 id="params_file-data">data</h4>

Data for $(location) expansions in args.

Defaults to `[]`

<h4 id="params_file-newline">newline</h4>

Line endings to use. One of ["auto", "unix", "windows"].

"auto" for platform-determined
"unix" for LF
"windows" for CRLF

Defaults to `"auto"`

<h4 id="params_file-kwargs">kwargs</h4>




