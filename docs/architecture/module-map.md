# Module Map

[Documentation Home](../README.md)

This page maps important files and directories to their roles in the current
repository.

## Top-Level Directories

| Path | Responsibility |
| --- | --- |
| `pdf2htmlEX/` | Converter source, CMake project, runtime assets, tests, and bundled third-party minifier jars. |
| `pdf2htmlEX/src/` | C++ and C implementation of the CLI and conversion pipeline. |
| `pdf2htmlEX/share/` | Manifest, CSS, JavaScript, and icon assets installed into the data directory. |
| `pdf2htmlEX/test/` | Python and shell tests, PDFs, reference HTML, and browser comparison tooling. |
| `buildScripts/` | Multi-stage shell scripts for dependency install, Poppler/FontForge builds, converter build, install, tests, and release artifacts. |
| `docker/` | Debian Bookworm Dockerfile that builds from a clone using the build scripts. |
| `.github/workflows/` | GitHub Actions workflow that runs `buildScripts/buildInstallLocallyApt` on Ubuntu 22.04. |
| `archive/` | Older Debian/PPA packaging files and Python 2 packaging scripts. |
| `poppler/` | Poppler source tree used by the local build scripts. |
| `fontforge/` | FontForge source tree used by the local build scripts. |
| `poppler-data/` | Poppler CMap and encoding data installed under the `pdf2htmlEX` data directory. |
| `patches/` | Historical FontForge patch files. Current `buildFontforge` has patch application commented out. |

## C++ And C Modules

| Path | Important symbols | Role |
| --- | --- | --- |
| `pdf2htmlEX/src/pdf2htmlEX.cc` | `main`, `parse_options`, `check_param`, `show_version_and_exit` | CLI entry point, option registration, PDF open, permission check, renderer invocation. |
| `pdf2htmlEX/src/Param.h` | `Param` | Central runtime configuration struct populated by `ArgParser`. |
| `pdf2htmlEX/src/ArgParser.*` | `ArgParser` | Thin `getopt_long` wrapper with typed option registration and positional arguments. |
| `pdf2htmlEX/src/Preprocessor.*` | `Preprocessor` | First Poppler pass. Collects used font code maps and max page dimensions. |
| `pdf2htmlEX/src/HTMLRenderer/HTMLRenderer.h` | `HTMLRenderer` | Main Poppler `OutputDev` implementation and central coordinator. |
| `pdf2htmlEX/src/HTMLRenderer/general.cc` | `HTMLRenderer::process`, `pre_process`, `post_process`, `dump_css`, `embed_file` | High-level page loop, output file setup, manifest application, CSS dumping, static asset embedding/linking. |
| `pdf2htmlEX/src/HTMLRenderer/state.cc` | `update*`, `check_state_change`, `prepare_text_line` | Tracks Poppler text/graphics state and converts it into HTML line/text states. |
| `pdf2htmlEX/src/HTMLRenderer/text.cc` | `drawString`, `is_char_covered` | Converts Poppler strings to `HTMLTextLine` content and coordinates covered-text behavior. |
| `pdf2htmlEX/src/HTMLRenderer/font.cc` | `install_font`, `dump_embedded_font`, `dump_type3_font`, `embed_font`, `export_remote_font` | Font discovery, extraction, reencoding, FontForge conversion, `@font-face` CSS generation. |
| `pdf2htmlEX/src/HTMLRenderer/image.cc` | `drawImage`, `drawSoftMaskedImage` | Records image regions for covered-text detection and delegates drawing to Poppler base behavior. Actual page images are produced by background renderers. |
| `pdf2htmlEX/src/HTMLRenderer/draw.cc` | `stroke`, `fill`, `eoFill`, transparency group callbacks | Records vector drawing operations for covered-text detection. |
| `pdf2htmlEX/src/HTMLRenderer/link.cc` | `processLink`, `get_linkaction_str` | Emits link overlay HTML and destination metadata. Remote GoTo and Launch actions are logged as TODO. |
| `pdf2htmlEX/src/HTMLRenderer/form.cc` | `process_form` | Emits basic text inputs and button/radio-like divs when `--process-form` is enabled. |
| `pdf2htmlEX/src/HTMLRenderer/outline.cc` | `process_outline`, `process_outline_items` | Emits nested outline HTML from Poppler outline items. |
| `pdf2htmlEX/src/BackgroundRenderer/BackgroundRenderer.*` | `BackgroundRenderer`, `getBackgroundRenderer` | Abstract interface and factory for page background renderers. |
| `pdf2htmlEX/src/BackgroundRenderer/SplashBackgroundRenderer.*` | `SplashBackgroundRenderer` | Poppler Splash-based PNG/JPEG page background rendering. |
| `pdf2htmlEX/src/BackgroundRenderer/CairoBackgroundRenderer.*` | `CairoBackgroundRenderer` | Poppler Cairo-based SVG background rendering and optional external JPEG extraction from SVG. |
| `pdf2htmlEX/src/HTMLTextPage.*` | `HTMLTextPage` | Per-page text line collection, clipping groups, text dumping. |
| `pdf2htmlEX/src/HTMLTextLine.*` | `HTMLTextLine`, `HTMLTextLine::State` | Per-line Unicode content, state span nesting, whitespace offsets, optional text optimization. |
| `pdf2htmlEX/src/HTMLState.h` | `FontInfo`, `HTMLTextState`, `HTMLLineState`, `HTMLClipState` | Lightweight state structures shared between renderer and text emitters. |
| `pdf2htmlEX/src/StateManager.h` | `StateManager`, `AllStateManager` | Reusable CSS class generation for dimensions, colors, transforms, spacing, and background sizes. |
| `pdf2htmlEX/src/CoveredTextDetector.*` | `CoveredTextDetector` | Determines whether character bounding boxes are covered by later non-text drawing. |
| `pdf2htmlEX/src/DrawingTracer.*` | `DrawingTracer` | Uses Cairo path math to track char, image, stroke, fill, clip, and CTM regions for covered-text detection. |
| `pdf2htmlEX/src/TmpFiles.*` | `TmpFiles` | Tracks temporary files for cleanup and size-limit checks. |
| `pdf2htmlEX/src/Base64Stream.*` | `Base64Stream` | Streams file contents as base64 for embedded HTML/CSS assets. |
| `pdf2htmlEX/src/StringFormatter.*` | `StringFormatter` | Reusable `vsnprintf` buffer helper used for filenames and formatted strings. |
| `pdf2htmlEX/src/util/ffw.*` | `ffw_*` | C ABI wrapper around FontForge internals. |
| `pdf2htmlEX/src/util/path.*` | `create_directories`, `sanitize_filename`, `get_suffix` | Output path helpers and split-page template sanitization. |
| `pdf2htmlEX/src/util/encoding.*` | `writeUnicodes`, `writeAttribute`, `writeJSON` | HTML/attribute/JSON escaping and UTF-8 output. |
| `pdf2htmlEX/src/util/const.*` | `EMBED_STRING_MAP`, `FORMAT_MIME_TYPE_MAP` | MIME types, embed/link wrappers, GB font-name workaround map. |
| `pdf2htmlEX/src/util/css_const.h.in` | `CSS::*` constants | CMake-generated CSS class-name constants shared with CSS and JS templates. |

## Python Modules And Scripts

| Path | Role |
| --- | --- |
| `pdf2htmlEX/test/test.py.in` | CMake template for common Python test utilities. It resolves the built binary path and `/tmp/pdf2htmlEX` test directories. |
| `pdf2htmlEX/test/test.py` | Generated local copy from the current build directory. |
| `pdf2htmlEX/test/test_output.py` | Non-browser unit tests. They verify conversion does not crash and expected output filenames are produced. |
| `pdf2htmlEX/test/browser_tests.py` | Base browser screenshot comparison suite using Selenium and Pillow. |
| `pdf2htmlEX/test/test_local_browser.py` | Firefox/WebDriver local browser implementation. |
| `pdf2htmlEX/test/test_remote_browser.py` | Sauce Labs remote browser implementation. It appears older and contains Python 2 idioms such as `sys.exc_clear()`. |
| `pdf2htmlEX/test/compareTestImages` | Python 3 helper that opens output, reference, and diff PNGs with `display`. |
| `pdf2htmlEX/test/regenerateTest` | Python 3 helper for copying a generated browser-test HTML file into the reference directory. |
| `archive/build_for_ppa.py` | Older Python 2 PPA source package script. |
| `archive/createDebianPackage` | Older Python 2 Debian package script. |

## Shell Scripts

`buildScripts/` contains the active build/release automation. Important entry
points are:

- `buildInstallLocallyApt`
- `buildInstallLocallyAlpine`
- `buildInstallLocallyDnf`
- `buildPdf2htmlEX`
- `buildPoppler`
- `buildFontforge`
- `getPoppler`
- `getFontforge`
- `installPdf2htmlEX`
- `runTests`
- `createAppImage`
- `createDebianPackage`
- `createImagesApt`
- `createImagesAlpine`

`pdf2htmlEX/share/build_css.sh` and `pdf2htmlEX/share/build_js.sh` minify CSS
and JavaScript through bundled YUI Compressor and Closure Compiler jars. If the
Java tool fails, each script falls back to copying the uncompressed input.
