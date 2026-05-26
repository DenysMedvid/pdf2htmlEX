# Contribution Notes

[Documentation Home](../README.md)

## General Approach

Make narrow changes and test the smallest behavior surface that changed. The
converter sits between Poppler internals, FontForge internals, browser layout,
and generated assets, so broad refactors can cause surprising regressions.

Before editing:

```sh
git status --short
```

After editing:

```sh
git diff --stat
git diff
```

## Coding Style Visible In Source

The C++ code uses:

- namespace `pdf2htmlEX`
- header/source pairs for most modules
- direct Poppler types in renderer signatures
- `std::unique_ptr` in newer code paths
- manual resource handling in C and some Cairo/Poppler paths
- `cerr` warnings for unsupported PDF features
- compact CSS class names generated through constants

The codebase mixes older and newer C++ styles because it has been carried
forward across Poppler and compiler changes.

## Generated Files

CMake configures several source-tree files:

- `pdf2htmlEX/src/pdf2htmlEX-config.h`
- `pdf2htmlEX/pdf2htmlEX.1`
- `pdf2htmlEX/src/util/css_const.h`
- `pdf2htmlEX/share/base.css`
- `pdf2htmlEX/share/fancy.css`
- `pdf2htmlEX/share/pdf2htmlEX.js`

Resource minification writes:

- `pdf2htmlEX/share/base.min.css`
- `pdf2htmlEX/share/fancy.min.css`
- `pdf2htmlEX/share/pdf2htmlEX.min.js`

Avoid accidental churn in generated files unless the behavior or templates
changed intentionally.

## Risky Areas

Font handling:
: `HTMLRenderer/font.cc` and `util/ffw.c` are tightly coupled to Poppler and
FontForge internals. Changes need PDFs with embedded fonts, external fonts,
CID fonts, bad ToUnicode maps, and Type 3 fonts when possible.

Text state and layout:
: `state.cc`, `text.cc`, `HTMLTextLine.cc`, and `StateManager.h` affect
positioning, selection, copy/paste, browser rendering, and file size.

Covered text detection:
: `DrawingTracer` and `CoveredTextDetector` implement patched behavior described
in the root README. Test with `--correct-text-visibility` modes and visual
browser tests.

Background rendering:
: Splash and Cairo renderers have different semantics. SVG output can fall back
to bitmap output. Image embedding choices affect output portability.

Manifest and runtime assets:
: `share/manifest`, CSS templates, and JavaScript viewer behavior determine how
generated pages load and display.

Build scripts:
: The scripts install packages, download large sources, and can remove/recreate
build directories. Review environment variables before running them.

## Review Checklist

- Does the change match the current Poppler/FontForge API used in this repo?
- Are CLI defaults and manpage text still consistent?
- Does `Param` include any new option or behavior state?
- Does generated HTML remain valid for embedded and non-embedded modes?
- Does split-page output still produce correct filenames?
- Are data-dir and manifest assumptions preserved?
- If font behavior changed, were embedded, external, and fallback cases
  considered?
- If text layout changed, were copy/paste and browser minimum font-size issues
  considered?
- If image/background behavior changed, were PNG/JPEG/SVG paths considered?
- Were fast tests run?
- Were browser tests run or explicitly skipped with a reason?
- Were docs updated if behavior changed?

## Test Expectations

At minimum, run the non-browser suite for output/file naming changes:

```sh
mkdir -p /tmp/pdf2htmlEX/tmp /tmp/pdf2htmlEX/dat /tmp/pdf2htmlEX/png /tmp/pdf2htmlEX/out /tmp/pdf2htmlEX/html
cd pdf2htmlEX/test
python3 test_output.py
```

For rendering, layout, forms, links, text visibility, fonts, or SVG/background
changes, run browser tests if the environment supports them:

```sh
cd pdf2htmlEX/test
./runLocalBrowserTests
```

Record failures honestly. The repository already documents a skipped
`text_visibility` browser test due clipping issues.

## Documentation Expectations

Update docs when changing:

- CLI options or defaults
- build scripts or dependencies
- output asset structure
- rendering pipeline behavior
- tests and test prerequisites
- supported platforms
- known limitations

For AI-agent work, update `llms.txt` and `llms-full.txt` only when the
high-level orientation or workflow changes.
