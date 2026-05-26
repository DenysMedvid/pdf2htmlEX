# Debugging

[Documentation Home](../README.md)

## Build Failures

Start with the scripted build stage that failed. The build scripts are designed
as separable steps, so rerun the smallest failing step instead of the full
top-level script.

Useful commands:

```sh
./buildScripts/reportEnvs
./buildScripts/buildPoppler
./buildScripts/buildFontforge
./buildScripts/buildPdf2htmlEX
```

Check whether `buildScripts/reSourceVersionEnvs` exists and matches the intended
versions:

```sh
sed -n '1,120p' buildScripts/reSourceVersionEnvs
```

For converter CMake failures, inspect:

- `pdf2htmlEX/CMakeLists.txt`
- `pdf2htmlEX/build/CMakeCache.txt`
- `pdf2htmlEX/build/CMakeFiles/CMakeConfigureLog.yaml`
- `poppler/build/CMakeCache.txt`
- `fontforge/build/CMakeCache.txt`

The current local build failed with CMake `4.3.0` before compilation because
the project declares a very old CMake minimum.

## Runtime Conversion Failures

Run with debug output and keep temporary files:

```sh
pdf2htmlEX/build/pdf2htmlEX --debug 1 --clean-tmp 0 --data-dir pdf2htmlEX/share input.pdf
```

Useful things to inspect:

- stderr warnings from Poppler, FontForge, and `HTMLRenderer`
- temporary directory path printed by `--debug 1`
- dumped fonts such as `f<id>.*`
- debug map files `f<id>.map`
- generated `__css`, `__pages`, and `__outline` temp files
- generated background assets

If conversion fails with "Cannot open the manifest file", pass a correct
`--data-dir` or install runtime assets.

If conversion fails on protected PDFs, check `--owner-password`,
`--user-password`, and `--no-drm`.

## Font Problems

Font issues usually pass through `HTMLRenderer/font.cc` and `util/ffw.c`.

Use:

```sh
--debug 1 --clean-tmp 0
```

Then inspect:

- raw extracted fonts: `__raw_font_<id>.*`
- converted fonts: `f<id>.<font-format>`
- mapping files: `f<id>.map`
- generated `@font-face` rules in CSS
- warnings about ToUnicode collisions, encoding conflicts, missing font files,
  unsupported writing mode, or Type 3 fallback

The signal handler has custom diagnostics for crashes while FontForge is
loading, generating, or saving a font.

The wiki crash guide recommends isolating FontForge failures by opening the last
mentioned temp font files in FontForge and using File -> Generate Fonts with
validation disabled. If FontForge crashes outside `pdf2htmlEX`, preserve the
font files and report the independent FontForge case.

For a quick view of input font metadata, use Poppler's `pdffonts`:

```sh
pdffonts input.pdf
```

Check the `emb`, `sub`, and `uni` columns before changing ToUnicode or external
font options.

## Text Placement Problems

Relevant files:

- `HTMLRenderer/state.cc`
- `HTMLRenderer/text.cc`
- `HTMLTextLine.cc`
- `StateManager.h`

Inspect generated HTML and CSS together. Text line divs use classes such as
`t`, `m*`, `x*`, `y*`, `h*`, `ff*`, `fs*`, `fc*`, `sc*`, `ls*`, and `ws*`.
The class definitions are in generated CSS.

Options that affect text placement and grouping include:

- `--zoom`
- `--fit-width`
- `--fit-height`
- `--font-size-multiplier`
- `--heps`
- `--veps`
- `--space-threshold`
- `--space-as-offset`
- `--optimize-text`
- `--tounicode`

## Covered Text Problems

Relevant files:

- `DrawingTracer.*`
- `CoveredTextDetector.*`
- `HTMLRenderer/text.cc`
- `HTMLRenderer/draw.cc`
- `HTMLRenderer/image.cc`
- background renderer classes

Options:

- `--correct-text-visibility 0`
- `--correct-text-visibility 1`
- `--correct-text-visibility 2`
- `--covered-text-dpi`
- `--proof`

Mode `1` handles fully covered text. Mode `2` also handles partially covered
text and can raise `actual_dpi` to `covered-text-dpi`, capped by `max_dpi`.

For visual investigation, compare text layer output and page background output.
The skipped `browser_tests/text_visibility` test is relevant but currently not
passing according to repository notes.

## Image And Background Problems

Relevant files:

- `BackgroundRenderer/BackgroundRenderer.*`
- `BackgroundRenderer/SplashBackgroundRenderer.*`
- `BackgroundRenderer/CairoBackgroundRenderer.*`
- `HTMLRenderer/image.cc`

Use `pdf2htmlEX -v` to confirm supported image formats.

Try switching:

```sh
--bg-format png
--bg-format jpg
--bg-format svg
```

For SVG output, check `--svg-node-count-limit` and `--svg-embed-bitmap`.
When external SVG JPEG extraction is enabled, look for `o<object-id>.jpg`
assets.

The wiki suggests increasing raster resolution for blurry output:

```sh
pdf2htmlEX --dpi 288 input.pdf
```

The older wiki command-line page used `--hdpi` and `--vdpi`; this source tree
registers `--dpi`.

## HTML/CSS Asset Debugging

Use non-embedded output to inspect separate files:

```sh
pdf2htmlEX/build/pdf2htmlEX \
  --data-dir pdf2htmlEX/share \
  --dest-dir out \
  --embed cfiJo \
  input.pdf
```

The `--embed` string uses lowercase to disable and uppercase to enable
embedding for CSS, fonts, images, JavaScript, and outline.

Check:

- main HTML shell from `share/manifest`
- generated CSS file
- font files `f<id>.<format>`
- background files `bg<page>.*`
- split page files when `--split-pages 1`

## Test Debugging

Fast suite:

```sh
cd pdf2htmlEX/test
python3 test_output.py -v
```

Browser suite:

```sh
cd pdf2htmlEX/test
./runLocalBrowserTests
```

After browser tests, inspect:

- `/tmp/pdf2htmlEX/png/*.out.png`
- `/tmp/pdf2htmlEX/png/*.ref.png`
- `/tmp/pdf2htmlEX/png/*.diff.png`
- `/tmp/pdf2htmlEX/html/*.out.html`
- `/tmp/pdf2htmlEX/html/*.ref.html`
- `/tmp/pdf2htmlEX/html/*.diff.html`

Use:

```sh
./compareTestImages <testName>
```

to open the three PNGs for a test.
