# Design Patterns

[Documentation Home](../README.md)

This page names patterns that are visible in the current code. The labels are
used as maintainability shorthand; they are not claims that the code is a
textbook implementation.

## Adapter: Poppler OutputDev To HTML

Where:

- `Preprocessor` in `pdf2htmlEX/src/Preprocessor.*`
- `HTMLRenderer` in `pdf2htmlEX/src/HTMLRenderer/`
- `SplashBackgroundRenderer` and `CairoBackgroundRenderer` in
  `pdf2htmlEX/src/BackgroundRenderer/`

Why:

Poppler exposes rendering through `OutputDev` virtual callbacks. The converter
adapts these callbacks to three different purposes: scanning font usage,
building HTML/CSS text output, and rendering background assets.

Tradeoffs:

This keeps PDF parsing inside Poppler, but the converter is tightly coupled to
Poppler APIs, data structures, and ABI changes. The build scripts explicitly
build a known Poppler version for this reason.

## Pipeline: Setup, Scan, Render, Assemble

Where:

- `main` and `HTMLRenderer::process`
- `HTMLRenderer::pre_process` and `post_process`
- `share/manifest`

Why:

Conversion is naturally staged: parse options, open PDF, scan, render pages,
dump CSS, then assemble final HTML. The first pass gives font subsetting and
zoom calculations information that is needed by the second pass.

Tradeoffs:

The two-pass pipeline can re-render pages and therefore costs time, especially
when background rendering is enabled. It avoids building a full document-wide
intermediate representation.

## Strategy And Factory: Background Rendering

Where:

- `BackgroundRenderer::getBackgroundRenderer`
- `BackgroundRenderer::getFallbackBackgroundRenderer`
- `SplashBackgroundRenderer`
- `CairoBackgroundRenderer`

Why:

Different output formats need different Poppler backends. Splash is used for
PNG/JPEG bitmaps. Cairo is used for SVG output. The renderer is chosen from
`--bg-format` and compile-time SVG support.

Tradeoffs:

The strategy boundary is narrow and useful. It still relies on `HTMLRenderer`
friend access for output streams, state managers, and formatting helpers, so the
background renderers are not independent modules.

## CSS Class Interning: Flyweight-Like State Managers

Where:

- `StateManager.h`
- `AllStateManager`
- `HTMLTextLine::State`

Why:

Many text fragments share font sizes, transforms, colors, positions, and
spacing. State managers intern these values and assign short CSS class ids.
HTML can then refer to classes instead of repeating inline styles.

Tradeoffs:

Generated HTML and CSS are compact and consistent. Debugging output requires
following class ids back to generated CSS. Floating-point epsilon merging can
affect whether values share a class.

## Builder/Template: Manifest-Based HTML Assembly

Where:

- `share/manifest`
- `HTMLRenderer::post_process`
- `HTMLRenderer::embed_file`

Why:

The final HTML document is assembled from literal manifest blocks, static data
files, generated CSS, generated pages, and generated outline HTML. This allows
the document shell and viewer assets to be customized through the data
directory.

Tradeoffs:

The manifest language is small and easy to inspect, but it is custom. Errors in
`data-dir` layout produce runtime failures such as "Cannot open the manifest
file".

## RAII And Scoped Cleanup

Where:

- `std::unique_ptr<PDFDoc>` and `globalParams`
- `TmpFiles` destructor
- output stream objects in `HTMLRenderer`
- `StringFormatter::GuardedPointer`

Why:

The code uses C++ object lifetimes to close files, clean temporary files, and
avoid some manual ownership mistakes.

Tradeoffs:

RAII is mixed with C libraries and Poppler/FontForge internals. `util/ffw.c`
uses process-global FontForge state and manual cleanup. `DrawingTracer` also
manages Cairo objects and manually allocated CTM stack entries, so changes in
these areas need careful ownership review.

## Callback Hooks: Covered Text Detection

Where:

- `DrawingTracer::on_char_drawn`
- `DrawingTracer::on_char_clipped`
- `DrawingTracer::on_non_char_drawn`
- lambdas in `HTMLRenderer` constructor

Why:

`DrawingTracer` should not own the policy for what to do with traced regions.
It reports drawing events, and `CoveredTextDetector` updates the per-character
coverage state.

Tradeoffs:

The callback boundary keeps tracing and policy separate. It also means text
visibility behavior depends on the order and consistency of several callbacks:
Poppler state updates, tracer events, detector state, and final text dumping.
