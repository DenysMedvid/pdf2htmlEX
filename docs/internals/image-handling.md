# Image Handling

[Documentation Home](../README.md)

The converter's image behavior is page-background oriented. It does not usually
export each PDF image as a separate HTML element from `drawImage`. Instead, it
renders non-text page content into a background layer and places selectable text
above it.

## Poppler Image Callbacks

`HTMLRenderer::drawImage` and `drawSoftMaskedImage` in
`pdf2htmlEX/src/HTMLRenderer/image.cc` call:

```cpp
tracer.draw_image(state);
```

then delegate to the base `OutputDev` implementation. The tracing call is used
for covered-text detection. Actual visible image output is produced later by a
background renderer at `endPage`.

## Background Renderers

`BackgroundRenderer::getBackgroundRenderer` chooses a renderer from
`--bg-format`:

- `png`: `SplashBackgroundRenderer` when Poppler has PNG support
- `jpg`: `SplashBackgroundRenderer` when Poppler has JPEG support
- `svg`: `CairoBackgroundRenderer` when SVG support is compiled in

`needNonText()` in `HTMLRenderer` returns true only when
`--process-nontext 1`.

## PNG/JPEG Backgrounds

`SplashBackgroundRenderer` renders each page through Poppler's
`SplashOutputDev` at `param.actual_dpi`.

It writes:

```text
bg<page-number>.png
bg<page-number>.jpg
```

to the temp directory when images are embedded, or to `dest-dir` when
`--embed-image 0`.

It then emits an `<img>` with classes for:

- background image marker (`bi`)
- left position
- bottom position
- width
- height

If embedding is enabled, the image is read back and written as a base64 data URI.

## SVG Backgrounds

`CairoBackgroundRenderer` renders each page through Poppler's `CairoOutputDev`
to:

```text
bg<page-number>.svg
```

It sets Cairo SVG version `1.2` and uses `param.actual_dpi` for fallback
resolution.

The emitted element is:

- `<img>` when bitmaps are embedded or no external bitmaps are used
- `<embed>` when SVG references external bitmaps

If `--embed-image 1`, the SVG is embedded as:

```text
data:image/svg+xml;base64,...
```

## SVG Bitmap Extraction

When `--svg-embed-bitmap 0`, `CairoBackgroundRenderer::setMimeData` may dump
some PDF image streams as external JPEG files instead of embedding them in SVG.

Conditions visible in code:

- PDF stream kind must be DCT/JPEG
- stream must have an object reference
- color space must be `DeviceRGB` or `DeviceGray`
- no `/Decode` array

Output file name:

```text
o<object-id>.jpg
```

The code intentionally avoids direct dumping of CMYK and other color spaces.

## SVG Node Count Fallback

If `--svg-node-count-limit` is non-negative, `CairoBackgroundRenderer` counts
`<` characters in the generated SVG as an approximate node count. If the count
exceeds the limit, it registers the SVG for cleanup and returns `false`.

`HTMLRenderer::endPage` then tries the fallback background renderer, which is
Splash for SVG configurations with a node limit.

## Covered Text Interaction

Background renderers draw text into the background layer when needed:

- proof mode
- fallback mode
- writing-mode fonts
- Type 3 fonts when not converted
- text rendering modes that use text as paths
- chars reported as covered by `CoveredTextDetector`

This is why image/background behavior is coupled to text visibility.

## Supported Formats

The existing local binary reports support for:

```text
png jpg svg
```

Run `pdf2htmlEX -v` for the actual binary being debugged.
