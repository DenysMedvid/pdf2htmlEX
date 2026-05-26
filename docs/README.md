# pdf2htmlEX Documentation

This documentation describes the repository state in this checkout. It is
written for human maintainers and AI agents that need to understand, build,
test, or modify the current patched `pdf2htmlEX` codebase.

The project wiki at <https://github.com/pdf2htmlEX/pdf2htmlEX/wiki> has been
read and localized into these docs. Wiki-derived pages are cross-checked against
this checkout where possible; historical or version-specific wiki notes are
marked as such.

`pdf2htmlEX` converts PDF files into HTML output. It keeps selectable text when
possible, exports CSS and web fonts for text layout, and renders non-text PDF
content into page background assets. The converter is mainly C++ with a small C
wrapper around FontForge internals, plus Python and shell tooling for tests,
packaging, and release workflows.

## Where To Start

Users who want to build the tool should start with
[Build Overview](build/overview.md), then
[Build From Source](build/build-from-source.md), and
[Dependencies](build/dependencies.md).

Developers new to the codebase should read
[Architecture Overview](architecture/overview.md),
[Rendering Pipeline](architecture/rendering-pipeline.md), and
[Codebase Tour](development/codebase-tour.md).

Maintainers should read
[Testing](development/testing.md),
[Contribution Notes](development/contribution-notes.md), and
[Troubleshooting](build/troubleshooting.md).

AI agents should read this index, then root-level [`llms.txt`](../llms.txt),
root-level [`llms-full.txt`](../llms-full.txt), and the internals page that
matches the intended change.

## User Guides

- [User Guide Index](user/README.md)
- [Quick Start](user/quick-start.md)
- [Prebuilt Packages And Docker](build/prebuilt-packages.md)
- [Optimization And Customization](user/optimization-and-customization.md)
- [Browser And Mobile Notes](user/browser-and-mobile.md)
- [FAQ And Limitations](user/faq-limitations.md)

## Architecture

- [Overview](architecture/overview.md)
- [Module Map](architecture/module-map.md)
- [Rendering Pipeline](architecture/rendering-pipeline.md)
- [Data Flow](architecture/data-flow.md)
- [Design Solutions](architecture/design-solutions.md)
- [Design Patterns](architecture/design-patterns.md)

## Build

- [Overview](build/overview.md)
- [Dependencies](build/dependencies.md)
- [Platforms](build/platforms.md)
- [Prebuilt Packages And Docker](build/prebuilt-packages.md)
- [Build From Source](build/build-from-source.md)
- [Troubleshooting](build/troubleshooting.md)

## Development

- [Getting Started](development/getting-started.md)
- [Codebase Tour](development/codebase-tour.md)
- [Testing](development/testing.md)
- [Debugging](development/debugging.md)
- [Contribution Notes](development/contribution-notes.md)

## Internals

- [CLI And Options](internals/cli-and-options.md)
- [PDF Processing](internals/pdf-processing.md)
- [Font Handling](internals/font-handling.md)
- [Image Handling](internals/image-handling.md)
- [CSS Generation](internals/css-generation.md)
- [HTML Generation](internals/html-generation.md)
- [Asset Output](internals/asset-output.md)
- [Python Tools](internals/python-tools.md)

## Reference

- [Glossary](reference/glossary.md)
- [Important Files](reference/important-files.md)
- [Configuration](reference/configuration.md)
- [Wiki Source Notes](reference/wiki-sources.md)

## Verification Snapshot

During the documentation pass, the existing built binary
`pdf2htmlEX/build/pdf2htmlEX` reported:

- `pdf2htmlEX` version `0.18.8.rc2`
- Poppler `24.06.1`
- FontForge date `20230101`
- Cairo `1.18.4`
- Supported image formats: `png`, `jpg`, `svg`

The non-browser output tests passed against that binary. Rebuilding the
existing CMake build directory failed before compilation because local CMake
`4.3.0` rejects the repository's `cmake_minimum_required(VERSION 2.6.0)`.
See [Testing](development/testing.md) and
[Build Troubleshooting](build/troubleshooting.md) for the command log.
