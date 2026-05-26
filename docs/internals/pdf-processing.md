# PDF Processing

[Documentation Home](../README.md)

PDF parsing and page interpretation are delegated to Poppler. The converter
implements Poppler `OutputDev` subclasses to receive rendering callbacks and
translate them into web output.

## Open And Validate PDF

`main` in `pdf2htmlEX/src/pdf2htmlEX.cc`:

1. Initializes `GlobalParams` with `param.poppler_data_dir`.
2. Builds optional `GooString` owner/user passwords.
3. Calls `PDFDocFactory().createPDFDoc`.
4. Checks `doc->isOk()`.
5. Checks `doc->okToCopy()` unless `--no-drm 1`.
6. Clamps `first_page` and `last_page`.
7. Calls `HTMLRenderer::process(doc.get())`.

## First Pass: Preprocessor

`Preprocessor` inherits from `OutputDev` and is used before actual conversion.

Important overrides:

- `useDrawChar()` returns `true`
- `needNonText()` returns `false`
- `drawChar(...)` records used codes by font
- `startPage(...)` records maximum page dimensions

It calls Poppler:

```cpp
doc->displayPage(this, i, DEFAULT_DPI, DEFAULT_DPI,
        0,
        (!(param.use_cropbox)),
        true,
        false,
        nullptr, nullptr, nullptr, nullptr);
```

The used-code maps feed the font subsetting/reencoding path.

## Second Pass: HTMLRenderer

`HTMLRenderer` also inherits from `OutputDev`. It requests non-text content when
`param.process_nontext` is true, but most non-text output is rendered later by
the selected background renderer.

During `HTMLRenderer::process`, each page is displayed through Poppler with
horizontal and vertical resolution equal to:

```text
text_zoom_factor() * DEFAULT_DPI
```

`DEFAULT_DPI` comes from project constants. `text_zoom_factor()` is
`text_scale_factor1 * text_scale_factor2`, where the factors are derived from
zoom/fit options and `--font-size-multiplier`.

## Page Lifecycle

`startPage`:

- resets covered-text state
- resets the drawing tracer
- stores current page number
- sets `HTMLTextPage` size
- resets renderer state

Poppler callbacks then update text/graphics state and draw content.

`endPage`:

- writes the page frame and content box
- renders and embeds/links the background layer
- dumps text HTML and generated per-page CSS hooks
- processes forms if enabled
- processes links
- writes page data used by JavaScript
- closes page markup
- writes split-page placeholder frame if `--split-pages 1`

## Page Boxes And DPI

`--use-cropbox 1` uses CropBox behavior through Poppler's display call. A false
value uses MediaBox behavior.

Before each page, `HTMLRenderer::process` computes:

```text
param.max_dpi = 72 * 9000 / max(page crop width, page crop height)
```

If `desired_dpi` is larger, `actual_dpi` is clamped. With
`--correct-text-visibility 2`, partially covered text can raise `actual_dpi` to
`--covered-text-dpi`, also capped by `max_dpi`.

## Links, Outlines, Annotations, Forms

Links:
: Poppler calls `processLink` for `AnnotLink` objects. The converter emits an
anchor and a positioned rectangle. Internal GoTo and URI actions are handled.
Remote GoTo and Launch actions are logged as TODO.

Outlines:
: After page rendering, `process_outline` walks Poppler outline items and emits
nested HTML lists.

Annotations:
: Background renderers pass an annotation callback controlled by
`param.process_annotation` to Poppler `displayPage`.

Forms:
: `process_form` uses Poppler form widgets for the current page and emits simple
HTML controls for text and button widgets.

## Unsupported Or Fallback PDF Features Visible In Code

- Writing-mode fonts are logged as unsupported and rendered as images.
- Type 3 fonts are rendered as images unless `--process-type3 1` and SVG support
  are available.
- Text rendering modes >= 4 are treated as image/fallback text.
- GoToR and Launch link actions are not implemented.
- Unsupported form field types print "Unsupported form field detected".
