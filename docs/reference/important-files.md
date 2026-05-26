# Important Files

[Documentation Home](../README.md)

| Path | Purpose | Why It Matters |
| --- | --- | --- |
| `README.md` | Root project overview and links. | First human-facing entry point and repository-specific covered-text notes. |
| `llms.txt` | Concise AI-agent guide. | Fast orientation for agents. |
| `llms-full.txt` | Fuller AI-agent guide. | Modification workflow and codebase map for agents. |
| `buildScripts/Readme.md` | Build-system explanation. | Describes why Poppler and FontForge are built as matched static dependencies. |
| `buildScripts/versionEnvs` | Build version variables. | Source of current Poppler, FontForge, and pdf2htmlEX version pins. |
| `buildScripts/buildInstallLocallyApt` | Main Apt build script. | Primary CI/build path. |
| `buildScripts/buildPdf2htmlEX` | Converter build script. | Recreates `pdf2htmlEX/build` and runs CMake/make. |
| `buildScripts/buildPoppler` | Poppler build script. | Configures static Poppler with features needed by converter. |
| `buildScripts/buildFontforge` | FontForge build script. | Configures static FontForge without GUI/Python extension. |
| `buildScripts/installPdf2htmlEX` | Install script. | Installs binary and poppler-data into `PDF2HTMLEX_PREFIX`. |
| `.github/workflows/build.yml` | GitHub Actions build. | Runs Apt build on Ubuntu 22.04. |
| `.travis.yml` | Older Travis config. | Records Linux/Bionic workflow and macOS/Windows support note. |
| `docker/Dockerfile.bookworm` | Debian Bookworm container build. | Containerized build path from remote clone. |
| `pdf2htmlEX/CMakeLists.txt` | Converter CMake project. | Defines sources, compile flags, dependencies, generated files, install rules, tests. |
| `pdf2htmlEX/pdf2htmlEX.1.in` | Manpage template. | User-facing option docs, though some option names lag source. |
| `pdf2htmlEX/src/pdf2htmlEX.cc` | Program entry point. | CLI parsing, PDF open, permissions, renderer invocation. |
| `pdf2htmlEX/src/Param.h` | Runtime configuration struct. | All options flow through this object. |
| `pdf2htmlEX/src/ArgParser.*` | CLI parser. | Typed wrapper around `getopt_long`. |
| `pdf2htmlEX/src/Preprocessor.*` | First PDF scan. | Collects used font codes and max page dimensions. |
| `pdf2htmlEX/src/HTMLRenderer/HTMLRenderer.h` | Main renderer declaration. | Central Poppler `OutputDev` adapter. |
| `pdf2htmlEX/src/HTMLRenderer/general.cc` | Main conversion loop and final assembly. | Coordinates preprocessing, page loop, CSS, manifest output. |
| `pdf2htmlEX/src/HTMLRenderer/state.cc` | State tracking. | Converts PDF graphics/text state to HTML/CSS line state. |
| `pdf2htmlEX/src/HTMLRenderer/text.cc` | Text conversion. | Converts Poppler strings to Unicode HTML text and offsets. |
| `pdf2htmlEX/src/HTMLRenderer/font.cc` | Font handling. | Highest-risk font extraction/reencoding/CSS output logic. |
| `pdf2htmlEX/src/HTMLRenderer/image.cc` | Image callback handling. | Traces image coverage; visible page images come from background renderers. |
| `pdf2htmlEX/src/HTMLRenderer/link.cc` | Link annotations. | Emits link overlays and destination metadata. |
| `pdf2htmlEX/src/HTMLRenderer/form.cc` | Form widgets. | Optional text/button form output. |
| `pdf2htmlEX/src/HTMLRenderer/outline.cc` | PDF outlines. | Emits sidebar outline HTML. |
| `pdf2htmlEX/src/BackgroundRenderer/` | Background renderer classes. | PNG/JPEG/SVG page background output and fallback. |
| `pdf2htmlEX/src/HTMLTextPage.*` | Per-page text storage. | Groups lines and clips before HTML output. |
| `pdf2htmlEX/src/HTMLTextLine.*` | Per-line text storage/output. | Emits line divs, spans, offsets, covered chars. |
| `pdf2htmlEX/src/StateManager.h` | CSS class interning. | Generates reusable CSS classes. |
| `pdf2htmlEX/src/CoveredTextDetector.*` | Covered-text policy. | Tracks whether chars are visually covered. |
| `pdf2htmlEX/src/DrawingTracer.*` | Coverage tracing. | Uses Cairo geometry for char/non-char coverage detection. |
| `pdf2htmlEX/src/util/ffw.*` | FontForge wrapper. | C bridge to FontForge internals. |
| `pdf2htmlEX/src/util/encoding.*` | HTML escaping. | Writes text, attributes, and JSON-like strings safely. |
| `pdf2htmlEX/src/util/path.*` | Path helpers. | Directory creation and split-page filename sanitization. |
| `pdf2htmlEX/share/manifest` | Final HTML template. | Controls document shell and asset insertion. |
| `pdf2htmlEX/share/base.css.in` | Core layout CSS template. | Required classes for pages, text, selection, print. |
| `pdf2htmlEX/share/fancy.css.in` | Optional viewer styling. | Sidebar/background/loading appearance. |
| `pdf2htmlEX/share/pdf2htmlEX.js.in` | Viewer JavaScript template. | Lazy loading, navigation, scaling behavior. |
| `pdf2htmlEX/test/test_output.py` | Fast tests. | Verifies conversions and output filenames. |
| `pdf2htmlEX/test/browser_tests.py` | Visual tests. | Selenium/Pillow screenshot comparison logic. |
| `pdf2htmlEX/test/README.md` | Test docs. | Records test commands and skipped/failing browser test note. |
| `archive/` | Historical packaging. | Useful context, but several scripts are Python 2-era. |
| `PopplerReleases.md` | Older Poppler release mapping. | Historical version mapping, not current build pin. |
