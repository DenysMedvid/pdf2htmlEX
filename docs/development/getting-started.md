# Getting Started

[Documentation Home](../README.md)

This repository is easiest to understand from the converter inward, then the
build scripts outward.

## First Files To Read

1. `README.md` for project-level notes and links.
2. `buildScripts/Readme.md` for why the build is script-driven.
3. `pdf2htmlEX/src/pdf2htmlEX.cc` for CLI parsing and program startup.
4. `pdf2htmlEX/src/HTMLRenderer/HTMLRenderer.h` for the central renderer API.
5. `pdf2htmlEX/src/HTMLRenderer/general.cc` for the document/page loop.
6. `pdf2htmlEX/src/HTMLRenderer/text.cc` and `state.cc` for text conversion.
7. `pdf2htmlEX/src/HTMLRenderer/font.cc` for font extraction/conversion.
8. `pdf2htmlEX/src/BackgroundRenderer/` for page background output.
9. `pdf2htmlEX/test/test_output.py` for fast non-browser behavior checks.

## Development Environment

The repository assumes a Linux-like development environment for building. The
active build scripts support Apt, Alpine, and DNF package managers, with Apt
used by GitHub Actions.

For code reading and test execution against the existing binary, useful tools
are:

- C++ compiler and CMake
- Python 3
- `ctest`
- `rg`
- Firefox, geckodriver, Xvfb, Selenium, and Pillow for browser tests

For full builds, use the build scripts instead of manually pointing CMake at
system Poppler/FontForge. The converter expects source/build trees in relative
locations.

## Local Binary And Tests

This checkout contains an executable binary at:

```text
pdf2htmlEX/build/pdf2htmlEX
```

It was usable during documentation verification. The non-browser test suite was
run directly:

```sh
mkdir -p /tmp/pdf2htmlEX/tmp /tmp/pdf2htmlEX/dat /tmp/pdf2htmlEX/png /tmp/pdf2htmlEX/out /tmp/pdf2htmlEX/html
cd pdf2htmlEX/test
python3 test_output.py
```

Result: 23 tests passed.

The CTest wrapper for the same basic suite was also run:

```sh
ctest --test-dir pdf2htmlEX/build -R test_basic --output-on-failure
```

Result: passed.

## Build Caution

A build verification command failed with local CMake `4.3.0` because the CMake
project declares an old minimum CMake version. See
[Build Troubleshooting](../build/troubleshooting.md).

If your task is not build-system maintenance, prefer targeted source changes and
test against a known-good existing binary or a build environment with compatible
CMake until the CMake policy issue is fixed.

## Before Changing Code

- Check `git status --short`.
- Identify whether a generated file is involved. CMake configures several files
  into the source tree.
- Read the relevant internals doc in `docs/internals/`.
- Inspect the nearest tests before changing behavior.
- Keep source changes narrow. Many modules are coupled to Poppler, FontForge,
  and browser layout behavior.

## Useful Commands

List source files:

```sh
rg --files pdf2htmlEX/src
```

Show CLI options in source:

```sh
rg -n "\.add\\(" pdf2htmlEX/src/pdf2htmlEX.cc
```

Run basic tests:

```sh
mkdir -p /tmp/pdf2htmlEX/tmp /tmp/pdf2htmlEX/dat /tmp/pdf2htmlEX/png /tmp/pdf2htmlEX/out /tmp/pdf2htmlEX/html
cd pdf2htmlEX/test
python3 test_output.py
```

List CTest tests:

```sh
ctest --test-dir pdf2htmlEX/build -N
```

Run converter with source data dir:

```sh
pdf2htmlEX/build/pdf2htmlEX --data-dir pdf2htmlEX/share input.pdf
```
