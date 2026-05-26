# Quick Start

[Documentation Home](../README.md)

This page localizes the project wiki's common `pdf2htmlEX` recipes and updates
the notes with behavior visible in this checkout.

## Before Converting

For better text rendering, especially with generated TrueType output on
Windows, the wiki recommends installing `ttfautohint` and passing it through:

```sh
pdf2htmlEX --external-hint-tool=ttfautohint input.pdf
```

The current source also supports `--auto-hint 1`, but external hinting takes
precedence when `--external-hint-tool` is set.

For CJK and other character-map-sensitive PDFs, make sure poppler-data is
available. Installed builds expect it under:

```text
<data-dir>/poppler
```

The path can be overridden:

```sh
pdf2htmlEX --poppler-data-dir /path/to/poppler-data input.pdf
```

If running an uninstalled build from the repository, pass the local data
directory:

```sh
pdf2htmlEX/build/pdf2htmlEX --data-dir pdf2htmlEX/share input.pdf
```

## Simple Conversion

```sh
pdf2htmlEX --zoom 1.3 pdf/test.pdf
```

This writes `test.html` to the current directory when no output filename is
given.

## Page Range And JPEG Backgrounds

```sh
pdf2htmlEX -f 3 -l 5 --fit-width 1024 --bg-format jpg pdf/test.pdf
```

This converts pages 3 through 5, scales each page so the width fits 1024
pixels, and asks the background renderer to write JPEG backgrounds. Supported
background formats are binary-dependent; this local binary reports `png`, `jpg`,
and `svg` from `pdf2htmlEX/build/pdf2htmlEX -v`.

## Publisher Output With Separate Assets

```sh
pdf2htmlEX --embed cfijo --dest-dir out pdf/test.pdf
```

The `--embed` string uses lowercase letters to disable embedding and uppercase
letters to enable embedding:

| Letter | Asset |
| --- | --- |
| `c`/`C` | CSS |
| `f`/`F` | fonts |
| `i`/`I` | images |
| `j`/`J` | JavaScript |
| `o`/`O` | outline |

The example writes `test.html` plus separate CSS, font, image, JavaScript, and
outline assets under `out/`. Separate files can be easier to cache on a web
server.

## Split Pages For Dynamic Loading

```sh
pdf2htmlEX \
  --embed cfijo \
  --split-pages 1 \
  --dest-dir out \
  --page-filename test-%d.page \
  pdf/test.pdf
```

This writes a main `test.html` plus one page fragment per page, such as
`test-1.page`, `test-2.page`, and so on. The main viewer can load pages
dynamically through the JavaScript shipped in the data directory.

The current filename logic accepts a `%d` placeholder with optional width and
zero padding, such as `page-%03d.page`.

## Fallback Mode

```sh
pdf2htmlEX --fallback 1 pdf/test.pdf
```

Fallback mode renders more page content into images and keeps hidden text for
search/copy where possible. It usually improves visual compatibility at the
cost of larger output and less native HTML text rendering.

## Useful Self-Inspection Commands

```sh
pdf2htmlEX --help
pdf2htmlEX -v
man pdf2htmlEX
```

The wiki command-line page is explicitly for version `0.12`; for this checkout,
prefer the local binary, `pdf2htmlEX/pdf2htmlEX.1.in`, and
[CLI And Options](../internals/cli-and-options.md).
