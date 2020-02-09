# setup-ruby

Note: this action used to be at `eregon/use-ruby-action` and was moved to the `ruby` organization.

This action downloads a prebuilt ruby and adds it to the `PATH`.

It is very efficient and takes about 5 seconds to download, extract and add the given Ruby to the `PATH`.
No extra packages need to be installed.

Compared to [actions/setup-ruby](https://github.com/actions/setup-ruby),
this actions supports many more versions and features.

### Supported Versions

This action currently supports these versions of MRI, JRuby and TruffleRuby:

| Interpreter | Versions |
| ----------- | -------- |
| Ruby | 2.2, 2.3.0 - 2.3.8, 2.4.0 - 2.4.9, 2.5.0 - 2.5.7, 2.6.0 - 2.6.5, 2.7.0, head |
| JRuby | 9.2.9.0 |
| TruffleRuby | 19.3.0, 19.3.1, head |
| Rubinius | 4.14 |

Note that Ruby ≤ 2.3 and the OpenSSL version it needs (1.0.2) are both end-of-life,
which means Ruby ≤ 2.3 is unmaintained and considered insecure.
On Windows, Ruby 2.4 uses OpenSSL 1.0.2, which is no longer maintained.
Ruby 2.2 resolves to 2.2.6 on Windows (last build from RubyInstaller) and 2.2.10 otherwise.

### Supported Platforms

The action works for all [GitHub-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners).

| Operating System | Versions |
| ----------- | -------- |
| Ubuntu  | `ubuntu-latest` (= `ubuntu-18.04`), `ubuntu-16.04` |
| macOS   | `macos-latest` |
| Windows | `windows-latest` |

Rubinius is only available on `ubuntu-latest`.

The prebuilt releases are generated by [ruby-builder](https://github.com/ruby/ruby-builder)
and on Windows by [RubyInstaller2](https://github.com/oneclick/rubyinstaller2).
`ruby-head` is generated by [ruby-dev-builder](https://github.com/ruby/ruby-dev-builder) and `truffleruby-head` is generated by [truffleruby-dev-builder](https://github.com/ruby/truffleruby-dev-builder).
The full list of available Ruby versions can be seen in [ruby-builder-versions.js](ruby-builder-versions.js)
for Ubuntu and macOS and in [windows-versions.js](windows-versions.js) for Windows.

## Usage

### Single Job

```yaml
name: My workflow
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.6
    - run: ruby -v
```

### Matrix

This matrix tests all stable releases of MRI, JRuby and TruffleRuby on Ubuntu and macOS.

```yaml
name: My workflow
on: [push]
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ ubuntu-latest, macos-latest ]
        ruby: [ 2.4, 2.5, 2.6, 2.7, jruby, truffleruby ]
    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: ${{ matrix.ruby }}
    - run: ruby -v
```

### Supported Version Syntax

* engine-version like `ruby-2.6.5` and `truffleruby-19.3.0`
* short version like `2.6`, automatically using the latest release matching that version (`2.6.5`)
* version only like `2.6.5`, assumes MRI for the engine
* engine only like `truffleruby`, uses the latest stable release of that implementation
* `.ruby-version` (also the default value) reads from the project's `.ruby-version` file

### Bundler

Currently, Bundler is guaranteed to be installed for all versions.
If the Ruby ships with Bundler (Ruby >= 2.6), that version is used.
Otherwise (Ruby < 2.6), Bundler 1 is installed when that Ruby was built.

### Caching `bundle install`

You can cache the installed gems with these two steps:

```yaml
    - uses: actions/cache@v1
      with:
        path: vendor/bundle
        key: bundle-use-ruby-${{ matrix.os }}-${{ matrix.ruby }}-${{ hashFiles('**/Gemfile.lock') }}
        restore-keys: |
          bundle-use-ruby-${{ matrix.os }}-${{ matrix.ruby }}-
    - name: bundle install
      run: |
        bundle config path vendor/bundle
        bundle install --jobs 4 --retry 3
```

When using a single job with a Ruby version, replace `${{ matrix.ruby }}` with the Ruby version.  
When using `.ruby-version`, replace `${{ matrix.ruby }}` with `${{ hashFiles('.ruby-version') }}`.

This uses the [cache action](https://github.com/actions/cache).
The code above is a more complete version of the [Ruby - Bundler example](https://github.com/actions/cache/blob/master/examples.md#ruby---bundler).
Make sure to include `use-ruby` in the `key` to avoid conflicting with previous caches.

## Windows

Note that running CI on Windows can be quite challenging if you are not very familiar with Windows.
It is recommended to first get your build working on Ubuntu and macOS before trying Windows.

* The default shell on Windows is not Bash but [PowerShell](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#using-a-specific-shell).
  This can lead issues such as multi-line scripts [not working as expected](https://github.com/ruby/setup-ruby/issues/13).
* The `PATH` contains [multiple compiler toolchains](https://github.com/ruby/setup-ruby/issues/19). Use `where` to debug which tool is used.
* MSYS2 is prepended to the `PATH`, similar to what RubyInstaller2 does.
* JRuby on Windows has a known bug that `bundle exec rake` [fails](https://github.com/ruby/setup-ruby/issues/18).

## Limitations

* This action currently only works with GitHub-hosted runners, not private runners.

## Versioning

This action follows semantic versioning with a moving `v1` branch.
This follows the [recommendations](https://github.com/actions/toolkit/blob/master/docs/action-versioning.md) of GitHub Actions.

## Credits

The current maintainer of this action is @eregon.
Most of the Windows logic is from https://github.com/MSP-Greg/actions-ruby by MSP-Greg.
Many thanks to MSP-Greg and Lars Kanis for the help with Ruby Installer.
