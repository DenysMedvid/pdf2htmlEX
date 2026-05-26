# Design Solutions

[Documentation Home](../README.md)

This page documents concrete design solutions visible in the current codebase.
It is different from [Design Patterns](design-patterns.md): patterns name code
shapes, while this page explains the engineering problems the repository solves,
where the solution lives, and the tradeoffs maintainers should keep in mind.

## Fixed-Layout HTML Instead Of Semantic Reflow

Problem:

PDF is a fixed-layout format, while HTML is naturally reflowable. Reconstructing
semantic paragraphs, lists, formulas, captions, code blocks, and hyphenation is
not reliable from visual PDF drawing commands alone.

Solution:

The converter emits absolutely positioned page content. Text lines are placed
with generated CSS classes, while non-text drawing is rendered into page
background assets. This preserves visual layout better than attempting semantic
HTML reconstruction.

Where:

- `pdf2htmlEX/src/HTMLTextPage.*`
- `pdf2htmlEX/src/HTMLTextLine.*`
- `pdf2htmlEX/src/HTMLRenderer/text.cc`
- `pdf2htmlEX/share/base.css.in`

Tradeoffs:

The output is visually faithful and text can often remain selectable, but it is
not semantic, not naturally reflowable, and can be heavy for mobile browsers.
This matches the wiki's stated limitation around reflowable text.

## Hybrid Text Layer Plus Background Layer

Problem:

PDF pages contain text, vector graphics, images, clipping, transparency, and
operations that do not map cleanly to browser-native text and CSS.

Solution:

`pdf2htmlEX` keeps native HTML text where possible and renders non-text content
into page backgrounds. In fallback or covered-text cases, the visual glyph may
come from the background while hidden or transparent text remains available for
selection/search.

Where:

- `pdf2htmlEX/src/HTMLRenderer/text.cc`
- `pdf2htmlEX/src/HTMLRenderer/draw.cc`
- `pdf2htmlEX/src/HTMLRenderer/image.cc`
- `pdf2htmlEX/src/BackgroundRenderer/`
- `pdf2htmlEX/src/CoveredTextDetector.*`

Tradeoffs:

This gives a practical balance between fidelity and selectable text. The hard
part is deciding when text is visually covered or clipped. The repository still
marks `browser_tests/text_visibility` skipped, so covered/clipped text remains a
risk area.

## Two-Pass PDF Processing

Problem:

Font subsetting, page fitting, and text output need information that is only
known after inspecting selected pages.

Solution:

Run a lightweight first pass with `Preprocessor`, then run the main rendering
pass with `HTMLRenderer`. The first pass records used character codes per font
and selected page extents; the second pass uses that information to emit fonts,
CSS, pages, and assets.

Where:

- `pdf2htmlEX/src/Preprocessor.*`
- `pdf2htmlEX/src/HTMLRenderer/general.cc`
- `pdf2htmlEX/src/pdf2htmlEX.cc`

Tradeoffs:

The code avoids a large document-wide intermediate representation, but selected
pages are traversed more than once. Any Poppler callback behavior used by both
passes must stay compatible.

## Poppler OutputDev As The Integration Boundary

Problem:

Implementing a PDF parser and renderer directly would be far beyond the scope
of this converter.

Solution:

Use Poppler for PDF loading, page interpretation, font discovery, and rendering
callbacks. Project-specific output devices adapt Poppler callbacks into scanning
and HTML generation.

Where:

- `Preprocessor`
- `HTMLRenderer`
- `SplashBackgroundRenderer`
- `CairoBackgroundRenderer`

Tradeoffs:

This delegates PDF semantics to Poppler, but makes the converter sensitive to
Poppler API and ABI changes. The build scripts therefore download and build a
specific Poppler source version.

## Font Extraction, Reencoding, And Web-Font Emission

Problem:

Selectable text requires browser fonts whose glyphs, widths, encodings, and
Unicode mappings match the PDF closely enough for layout and copy/paste.

Solution:

Extract embedded fonts or locate external fonts through Poppler, process them
with the FontForge wrapper, reencode used glyphs, fix metrics, optionally hint,
then emit `@font-face` CSS and font files or data URIs.

Where:

- `pdf2htmlEX/src/HTMLRenderer/font.cc`
- `pdf2htmlEX/src/util/ffw.c`
- `pdf2htmlEX/src/util/ffw.h`
- `pdf2htmlEX/src/Preprocessor.*`

Tradeoffs:

The approach preserves visual text better than relying on browser substitution,
but it is tightly coupled to FontForge internals. Font conversion is one of the
most failure-prone areas, especially for malformed fonts, ToUnicode collisions,
CID fonts, and Type 3 fonts.

## Embed External Fonts By Default

Problem:

PDF files may reference fonts that are not embedded. PDF viewers have standard
font assumptions and substitution behavior, but browsers do not share the same
PDF font environment or metrics.

Solution:

The default `--embed-external-font 1` embeds a local matched font when Poppler
can locate one. If disabled or unavailable, the converter emits CSS using the
font name plus a generic family fallback.

Where:

- `HTMLRenderer/font.cc`
- `util/const.cc`
- `Param.h`

Tradeoffs:

Embedding external fonts favors visual fidelity for users who do not know which
fonts are safe to omit. It can greatly increase output size and may be
undesirable for publishing workflows that control client fonts.

## CSS Class Interning Instead Of Inline Styles

Problem:

Directly writing every transform, position, color, font size, and spacing value
inline would make generated HTML large and repetitive.

Solution:

The converter interns repeated state values into generated CSS classes. Page
HTML refers to compact classes such as font, size, transform, x/y position, and
spacing classes.

Where:

- `pdf2htmlEX/src/StateManager.h`
- `pdf2htmlEX/src/HTMLRenderer/state.cc`
- `pdf2htmlEX/src/HTMLTextLine.cc`

Tradeoffs:

Output is smaller and more regular. Debugging requires following class names
back to generated CSS, and floating-point epsilon decisions can affect class
reuse.

## Manifest-Based HTML Assembly

Problem:

The converter needs a final HTML shell, default CSS, optional viewer
JavaScript, generated CSS, page content, outline content, and optionally linked
or embedded assets.

Solution:

Use `data-dir/manifest` as a small template language. The manifest combines
literal HTML, static files, and generated insertions such as `$css`, `$pages`,
and `$outline`.

Where:

- `pdf2htmlEX/share/manifest`
- `HTMLRenderer::post_process`
- `HTMLRenderer::embed_file`

Tradeoffs:

Users can customize the document shell without editing C++ source. The manifest
syntax is custom and documented by the wiki as experimental, so custom data
directories should be tested after converter changes.

## Embedded Or Separate Asset Output

Problem:

Different use cases want different packaging: a single portable HTML file, or
separate cacheable assets for web publishing.

Solution:

Expose embedding controls through `--embed`, `--embed-css`, `--embed-font`,
`--embed-image`, `--embed-javascript`, and `--embed-outline`. The same generated
content can be written to temp files for embedding or to `dest-dir` for linking.

Where:

- `pdf2htmlEX/src/pdf2htmlEX.cc`
- `HTMLRenderer/general.cc`
- `HTMLRenderer/font.cc`
- `BackgroundRenderer/*`
- `share/manifest`

Tradeoffs:

Single-file output is convenient but larger because binary assets become
base64. Separate files improve caching and inspection but require a directory of
assets to stay together.

## Background Renderer Selection And Fallback

Problem:

PNG/JPEG and SVG backgrounds require different rendering backends. SVG output
can also become too complex for browsers or include bitmap cases that need
special handling.

Solution:

Use a background-renderer strategy selected by `--bg-format`. Splash renders
PNG/JPEG. Cairo renders SVG when SVG support is compiled in. SVG node count can
trigger fallback to bitmap output.

Where:

- `pdf2htmlEX/src/BackgroundRenderer/BackgroundRenderer.*`
- `SplashBackgroundRenderer.*`
- `CairoBackgroundRenderer.*`
- `HTMLRenderer/general.cc`

Tradeoffs:

The option exposes useful output choices without changing the main renderer.
SVG support depends on Cairo and compile-time configuration, while SVG fallback
behavior means output format can vary per page.

## Pinned Static Dependency Build

Problem:

The converter uses Poppler and FontForge APIs that change across releases and
are not always exposed as stable distribution packages.

Solution:

Build scripts download specific Poppler, FontForge, and poppler-data versions,
compile static libraries, and link the converter against those known trees.

Where:

- `buildScripts/versionEnvs`
- `buildScripts/getPoppler`
- `buildScripts/buildPoppler`
- `buildScripts/getFontforge`
- `buildScripts/buildFontforge`
- `buildScripts/buildPdf2htmlEX`

Tradeoffs:

Builds are more reproducible with respect to these dependencies, but the build
process is heavier than a normal system-library CMake project. Modern CMake
compatibility also becomes a separate maintenance concern in this checkout.

## Optional Default Viewer Rather Than Full UI Product

Problem:

Generated output needs basic navigation, lazy page loading, outline handling,
and link behavior, but a full PDF-viewer UI would be a separate product.

Solution:

Ship default viewer assets in the data directory and let the manifest include
them. Publishers can customize or replace those assets.

Where:

- `pdf2htmlEX/share/pdf2htmlEX.js.in`
- `pdf2htmlEX/share/base.css.in`
- `pdf2htmlEX/share/fancy.css.in`
- `pdf2htmlEX/share/manifest`

Tradeoffs:

The repository provides a usable default shell while keeping UI scope limited.
The wiki's "unfeatures" page explicitly treats full viewer UI features as
outside the converter's core scope.
