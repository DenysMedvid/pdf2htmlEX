# Asset Output

[Documentation Home](../README.md)

Output files depend on `--embed-*`, `--split-pages`, `--bg-format`, and font
options. This page describes the visible structure from current code.

## Main HTML

The main HTML file is always written to:

```text
<dest-dir>/<output-filename>
```

If no output filename is provided:

- `input.pdf` becomes `input.html`
- otherwise the input filename gets `.html` appended

## CSS

Generated PDF-specific CSS is written to:

- temp `__css` when `--embed-css 1`
- `<dest-dir>/<css-filename>` when `--embed-css 0`

Default CSS filename:

- `input.pdf` -> `input.css`
- otherwise `<input>.css`

Static CSS files are read from `--data-dir` through `share/manifest`:

- `base.min.css`
- `fancy.min.css`

They are embedded or linked according to `--embed-css`.

## Fonts

Generated web fonts are named:

```text
f<font-id>.<font-format>
```

They are written to:

- temp directory when `--embed-font 1`
- `dest-dir` when `--embed-font 0`

Supported output format strings visible in `font.cc` and MIME mappings:

- `ttf`
- `otf`
- `woff`
- `eot`
- `svg`

The default in source is `woff`.

## Background Images

Splash PNG/JPEG backgrounds:

```text
bg<page-number>.png
bg<page-number>.jpg
```

Cairo SVG backgrounds:

```text
bg<page-number>.svg
```

SVG external JPEG streams, when dumped:

```text
o<object-id>.jpg
```

These files are written to the temp directory when `--embed-image 1`, or to
`dest-dir` when `--embed-image 0`.

## Outline

Generated outline HTML is written to:

- temp `__outline` when `--embed-outline 1`
- `<dest-dir>/<outline-filename>` when `--embed-outline 0`

Default outline filename:

- `input.pdf` -> `input.outline`
- otherwise `<input>.outline`

If `--process-outline 0`, outline processing is skipped.

## Split Page Files

When `--split-pages 1`, page content files are generated from
`--page-filename`.

Default:

```text
<input-without-pdf-suffix>%d.page
```

Examples from the test suite:

- `--split-pages 1 3-pages.pdf` -> `3-pages1.page`, `3-pages2.page`,
  `3-pages3.page`
- `--page-filename foo.xyz` -> `foo1.xyz`, `foo2.xyz`
- `--page-filename fo%03do.xyz` -> `fo001o.xyz`, `fo002o.xyz`

`sanitize_filename` only treats `%d` with optional digits as a page placeholder.

## JavaScript And Static Assets

Manifest-referenced runtime assets:

- `compatibility.min.js`
- `pdf2htmlEX.min.js`
- `pdf2htmlEX-64x64.png`

JavaScript embedding is controlled by `--embed-javascript`. The loading icon is
controlled by `--embed-image`.

## Manifest Customization

The wiki describes `data-dir/manifest` as the template for final output. In this
checkout, the source template is `pdf2htmlEX/share/manifest`.

Manifest commands visible in the file:

| Prefix | Meaning |
| --- | --- |
| `#` | comment |
| `@` | include a data-dir file, embedded or linked according to `--embed-*` |
| `$` | insert generated converter content |
| `"""` | copy a literal block through to output |

Important generated insertions are `$css`, `$outline`, and `$pages`.

The wiki recommends customizing optional theme/UI assets rather than required
layout assets:

- `base.css.in`: required rendering/layout CSS
- `fancy.css.in`: optional visual theme
- `pdf2htmlEX.js.in`: default/demonstration viewer behavior

Static resources can be hosted at stable URLs by using a custom manifest, while
document-specific generated CSS and page content remain per conversion.

## Temporary Files

`HTMLRenderer` and font/background code create temporary files under a unique
`pdf2htmlEX-XXXXXX` directory. Examples:

- `__css`
- `__pages`
- `__outline`
- raw dumped fonts `f<id>.<suffix>`
- converted fonts `f<id>.<font-format>`
- debug raw fonts `__raw_font_<id>.<suffix>`
- debug maps `f<id>.map`
- Type 3 glyph SVGs `f<id>-<code>.svg`
- background images `bg<page>.*`

`TmpFiles` removes registered temp files and the temp directory when
`--clean-tmp 1`. With `--clean-tmp 0`, temp files are kept for debugging.

## Size Limit

`--tmp-file-size-limit <KB>` checks the registered temporary file total after
each page. If the total exceeds the limit, processing stops before the next
page. `-1` means no limit.
