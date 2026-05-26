# Optimization And Customization

[Documentation Home](../README.md)

This page localizes the wiki's output customization and optimization guidance
and ties it to current repository files.

## Data Directory And Manifest

The output shell is controlled by the data directory. Run:

```sh
pdf2htmlEX -v
```

to see the compiled default data directory for a binary, or override it:

```sh
pdf2htmlEX --data-dir /path/to/data-dir input.pdf
```

The main template is:

```text
<data-dir>/manifest
```

In this checkout, the source template is:

```text
pdf2htmlEX/share/manifest
```

The manifest syntax is line-oriented:

| Prefix | Meaning |
| --- | --- |
| `#` | comment |
| `@` | embed or link a data-dir file according to `--embed-*` options |
| `$` | special generated content inserted by `pdf2htmlEX` |
| `"""` | begin or end a literal block copied into the output |

Important manifest placeholders:

| Placeholder | Inserted Content |
| --- | --- |
| `$css` | generated PDF-specific CSS |
| `$outline` | generated outline HTML |
| `$pages` | generated page content |

Manifest-referenced static assets in this checkout include:

- `base.min.css`
- `fancy.min.css`
- `compatibility.min.js`
- `pdf2htmlEX.min.js`
- `pdf2htmlEX-64x64.png`

The wiki says manifest syntax is experimental. Treat custom manifests as part of
your deployment and test them after converter upgrades.

## CSS And UI Customization

The wiki recommends changing the optional theme layer rather than the required
layout layer:

- `base.css.in`: required layout and rendering rules; modify with caution
- `fancy.css.in`: optional default visual theme; safer place for appearance
  changes
- custom CSS: can be included after `$css` in a custom manifest if the site
  needs to override generated PDF styles

The JavaScript viewer is a demonstration/default viewer, not the only possible
UI. Sites can provide their own UI around the generated page structure.

## Embedding Tradeoffs

Assets can be embedded as data URIs or emitted as separate files. Embedding
reduces HTTP requests, but base64 data is larger than the original binary data.
The wiki uses the rule of thumb that base64 adds roughly one third.

For many published documents, separate static assets can improve caching:

```sh
pdf2htmlEX --embed cfijo --dest-dir out input.pdf
```

For single-file sharing, use embedded assets:

```sh
pdf2htmlEX input.pdf
```

## Static Resource Caching

The wiki notes that files such as `base.css`, `fancy.css`, and
`pdf2htmlEX.js` are static. A publishing system can store equivalent assets at
stable URLs and reference them from a custom manifest so browsers cache them
across many converted documents.

When doing this, keep generated PDF-specific CSS (`$css`) document-specific.

## HTTP Compression

PDF has built-in compression; HTML does not. The wiki recommends enabling HTTP
compression, such as gzip or deflate, on the web server. Gzipped generated HTML
can be smaller over the network than the original uncompressed HTML file.

## Image Optimization

The wiki suggests optimizing PNG output with tools such as `pngnq` and
`pngcrush`.

The wiki footer also contains an unverified WebP workflow contributed as a wiki
note:

1. Convert `bg*.png` files to WebP with ImageMagick `convert`.
2. Rewrite split page files from `.png` to `.webp`.
3. Optionally replace external WebP references with base64 data URIs.

That workflow is not implemented by `pdf2htmlEX` in this checkout and was not
tested during the documentation pass. If used, apply it after conversion and
verify every generated page in target browsers.

## Optimize PDFs Before Conversion

The wiki says `pdf2htmlEX` is a converter, not a general PDF optimizer.
Pre-processing may reduce generated output size when the input PDF contains
unused objects, deleted objects, duplicate fonts, unnecessary annotations, or
large images.

One wiki example uses Ghostscript:

```sh
gs -sDEVICE=pdfwrite -sOutputFile=output.pdf -dNOPAUSE -dBATCH input.pdf
```

Ghostscript and similar tools have many options and tradeoffs; test optimized
PDFs visually before using them as conversion input.

## Font-Related Size Choices

The wiki highlights external fonts as a common size tradeoff. By default this
converter embeds local matches for fonts that were not embedded in the PDF:

```sh
--embed-external-font 1
```

Disabling that behavior can reduce output size:

```sh
pdf2htmlEX --embed-external-font 0 input.pdf
```

The risk is that browser-side font substitution may have different metrics,
causing layout drift.
