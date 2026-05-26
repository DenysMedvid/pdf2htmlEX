# Configuration

[Documentation Home](../README.md)

Configuration exists at several layers: build scripts, CMake, runtime CLI,
environment variables, data-directory templates, and test settings.

## Build Script Configuration

`buildScripts/versionEnvs` exports:

- `PDF2HTMLEX_VERSION`
- `POPPLER_VERSION`
- `FONTFORGE_VERSION`
- `LINUX_DEPLOY_URL`
- `PDF2HTMLEX_BRANCH`
- `BUILD_OS`
- `BUILD_DIST`
- `BUILD_DATE`
- `BUILD_TIME`
- `MACHINE_ARCH`
- `PDF2HTMLEX_NAME`

It writes those values to `buildScripts/reSourceVersionEnvs`, which later build
scripts source.

Top-level local build scripts set:

- `UNATTENDED`
- `MAKE_PARALLEL`
- `PDF2HTMLEX_BRANCH`
- `PDF2HTMLEX_PREFIX`
- `DEBIAN_FRONTEND`

`travisLinuxDoItAll` also sets:

- `PDF2HTMLEX_PATH`
- `PDF2HTMLEX_BRANCH=$TRAVIS_BRANCH`

Release/upload scripts use variables such as:

- `DOCKER_HUB_USERNAME`
- `DOCKER_HUB_PASSWORD`
- `GITHUB_USERNAME`
- `GITHUB_TOKEN`
- `TRAVIS_REPO_SLUG`

## CMake Configuration

`pdf2htmlEX/CMakeLists.txt` defines:

- default `CMAKE_BUILD_TYPE=Release`
- `ENABLE_SVG` option, default `ON`
- `CMAKE_EXPORT_COMPILE_COMMANDS ON`
- `CMAKE_CXX_STANDARD 20`
- generated config/manpage/CSS/JS files

It reads `PDF2HTMLEX_VERSION` from the shell environment:

```cmake
set(PDF2HTMLEX_VERSION $ENV{PDF2HTMLEX_VERSION})
```

It sets install locations for:

- binary: `bin`
- runtime resources: `share/pdf2htmlEX`
- manpage: `share/man/man1`

The existing local build cache uses:

```text
CMAKE_INSTALL_PREFIX=/home/dmedvid/work/pdf2htmlEx-install
ENABLE_SVG=ON
```

## Runtime CLI Configuration

Runtime behavior is primarily configured through CLI options. See
[CLI And Options](../internals/cli-and-options.md) for the complete option map.

Major configuration groups:

- page range
- zoom/fit/dpi
- embedding and output filenames
- non-text, outlines, annotations, forms, printing, fallback
- font conversion and hinting
- text spacing, Unicode, optimization, covered-text behavior
- background format and SVG behavior
- PDF passwords and DRM override
- temp/data/poppler-data directories
- debug/proof/quiet modes

## Runtime Environment Variables

Visible in source:

`TMPDIR`
: Default temp root before `--tmp-dir`.

`APPDIR`
: AppImage runtime adjustment. If set, `data_dir` becomes `APPDIR + data_dir`
before CLI overrides are applied.

Visible in build/test scripts:

`P2H_TEST_GEN`
: Enables browser reference-generation mode in tests.

`P2H_TEST_REMOTE`
: Mentioned in old test README content for remote tests.

`SAUCE_USERNAME`, `SAUCE_ACCESS_KEY`
: Credentials for `test_remote_browser.py`.

`PDF2HTMLEX_PATH`
: Used by shell test helpers and build scripts to locate the binary.

`PDF2HTMLEX_DATDIR`, `PDF2HTMLEX_TMPDIR`, `PDF2HTMLEX_PREDIR`,
`PDF2HTMLEX_TEST_DIR`
: Used by `produceHtmlForBrowserTests`.

## Data Directory Configuration

The runtime data directory contains:

- `manifest`
- `base.css` and `base.min.css`
- `fancy.css` and `fancy.min.css`
- `pdf2htmlEX.js` and `pdf2htmlEX.min.js`
- `compatibility.js` and `compatibility.min.js`
- `pdf2htmlEX-64x64.png`
- `LICENSE`
- `poppler/` data after install

`--data-dir` points to this directory. `--poppler-data-dir` can separately point
to Poppler data.

The manifest controls final HTML assembly. Custom data directories must contain
all files referenced by their manifest.

The project wiki emphasizes that custom output starts with the data directory:
copy the default data directory, edit `manifest` or optional assets, and pass
the copy with `--data-dir`.

The manifest syntax visible in `pdf2htmlEX/share/manifest` is:

| Prefix | Meaning |
| --- | --- |
| `#` | comment |
| `@` | data-dir asset, embedded or linked according to options |
| `$` | generated converter insertion |
| `"""` | literal HTML block |

The wiki labels the manifest syntax experimental, so custom manifests should be
tested after converter upgrades.

## Test Configuration

CMake configures these paths into `test/test.py`:

- `PDF2HTMLEX_PATH`
- `PDF2HTMLEX_TMPDIR`
- `PDF2HTMLEX_DATDIR`
- `PDF2HTMLEX_PNGDIR`
- `PDF2HTMLEX_OUTDIR`
- `PDF2HTMLEX_HTMDIR`
- `PDF2HTMLEX_PREDIR` if configured

The generated local `test.py` currently points to `/tmp/pdf2htmlEX/*`
directories.

Browser tests use fixed browser dimensions:

```text
800 x 1200
```

## Packaging Configuration

`archive/debian/control` and `buildScripts/PKGBUILD` contain packaging metadata.
The Arch `PKGBUILD` versions are older than the active `versionEnvs` values.

`buildScripts/createDebianPackage` creates package control data dynamically from
build variables and Git config.
