# FAQ And Limitations

[Documentation Home](../README.md)

This page localizes the project wiki FAQ, limitations, and "unfeatures" pages.
It is user-facing; implementation details live in the internals docs.

## What Languages And Libraries Are Involved?

The wiki describes the project as a mix of:

- C for the FontForge wrapper
- C++ for the converter
- CSS and HTML for output
- JavaScript for the optional viewer UI
- Java for optional CSS/JavaScript minification
- Python and shell scripts for tests and tooling

The main libraries named by the wiki are Poppler, FontForge, jQuery, and
Closure Compiler. In this checkout, `compatibility.min.js` is derived from
PDF.js compatibility code, and Java minification falls back to copying original
assets if minification fails.

## A PDF Does Not Convert Correctly

Possible causes listed by the wiki:

- the file does not follow the PDF standard, even if some viewers display it
- a Poppler or FontForge issue
- a `pdf2htmlEX` limitation
- an unsupported or difficult PDF feature

Useful first checks:

```sh
pdf2htmlEX -v
pdf2htmlEX --debug 1 --clean-tmp 0 input.pdf
```

If the issue is visual, try:

```sh
pdf2htmlEX --fallback 1 input.pdf
```

If text copy/paste is wrong, verify whether copy/paste works in a PDF viewer.
If the PDF viewer cannot extract the intended text, `pdf2htmlEX` usually cannot
invent it.

## Cannot Open The Manifest File

The wiki answer is to install the program. In this checkout, the concrete cause
is that `share/manifest` is read from `--data-dir`.

For an uninstalled local binary:

```sh
pdf2htmlEX/build/pdf2htmlEX --data-dir pdf2htmlEX/share input.pdf
```

For installed builds, verify the installed data directory with:

```sh
pdf2htmlEX -v
```

## Generated HTML Freezes Firefox

Wiki suggestions:

- do not zoom in too much
- lower `--font-size-multiplier`

Those options can greatly increase page dimensions or text scale, which may
stress browser layout and painting.

## Generated HTML Looks Bad

Check browser support first. See [Browser And Mobile Notes](browser-and-mobile.md).

Then isolate the layer:

- `--fallback 1`: tests image-heavy compatibility output
- `--process-nontext 0`: removes non-text background rendering
- `--embed cfijo`: emits separate files for inspection
- `--debug 1 --clean-tmp 0`: keeps intermediate files

## Generated HTML Is Too Large

Wiki causes and mitigations:

- embedded data URIs increase binary asset size because of base64 encoding
- separate assets can be enabled with `--embed`
- external font embedding can be disabled with `--embed-external-font 0`
- HTTP compression can reduce transferred HTML size
- input PDFs can be optimized before conversion

See [Optimization And Customization](optimization-and-customization.md).

## Text Is Correct But Hard To Read

Wiki suggestions:

```sh
pdf2htmlEX --external-hint-tool=ttfautohint input.pdf
pdf2htmlEX --auto-hint 1 input.pdf
```

The external hint tool is preferred when available. `--auto-hint` uses
FontForge hinting.

## Copy/Paste Text Is Wrong

Try:

```sh
pdf2htmlEX --tounicode 1 input.pdf
```

If that hurts visual output, compare with:

```sh
pdf2htmlEX --tounicode -1 input.pdf
```

Mode `0` is the default auto mode. See
[Font Handling](../internals/font-handling.md) for why ToUnicode maps can be
useful, missing, or wrong.

## No Images Are Generated

Wiki checks:

- do not set `--process-nontext 0` if image/background output is needed
- make sure image libraries were available when Poppler was compiled

In this checkout, the local binary reports image formats with:

```sh
pdf2htmlEX -v
```

## Text Or Images Are Blurry

Wiki suggestions:

```sh
pdf2htmlEX --zoom 2 input.pdf
pdf2htmlEX --dpi 288 input.pdf
```

The old wiki command-line page uses `--hdpi` and `--vdpi`; the current source
registers `--dpi`.

## Known Limitations

The wiki lists these difficult areas:

- reflowable text
- stroked text with separate fill and stroke colors
- clipped or overlapped text
- vertical or nonstandard writing-mode fonts
- browser font-size rounding

Current source includes `--correct-text-visibility` to handle some occluded text
cases, but the repository also skips the `browser_tests/text_visibility` test
because of clipping issues. Treat clipped/covered text as a known risk area.

## Reflowable Text

The wiki explains that PDF is primarily a fixed-layout format. Reflowing text
requires reconstructing paragraphs, indentation, line heights, bullets,
hyphenation, code blocks, formulas, underlines, and other semantics from visual
positions. `pdf2htmlEX` is designed for high-fidelity fixed-layout output, not
semantic reflow.

## Deliberately Unsupported Scope

The wiki's "unfeatures" page says the converter generally should not become a
general PDF manipulation or optimization tool. Examples outside the core scope:

- PDF linearization
- general font optimization
- general image optimization
- full PDF-viewer UI features

The converter does handle format-conversion needs such as page scaling,
rasterizing non-HTML-friendly PDF drawing operations, and text output
optimization.

The default `pdf2htmlEX.js` is a demonstration/default viewer for generated
pages. Publishers who need a richer UI are expected to build one around the
generated structure.
