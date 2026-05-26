# Python Tools

[Documentation Home](../README.md)

Python is used for tests and historical packaging helpers. It is not used in
the converter runtime.

## Test Helper Template

`pdf2htmlEX/test/test.py.in` is configured by CMake into
`pdf2htmlEX/test/test.py`.

It defines `Common`, which:

- locates the built `pdf2htmlEX` executable
- defines source/test/data directories
- defines `/tmp/pdf2htmlEX` test directories
- filters `share/manifest` by removing blocks between `#TEST_IGNORE_BEGIN` and
  `#TEST_IGNORE_END`
- copies `base.min.css` and test `fancy.min.css` into the temporary data dir
- runs the converter with `--data-dir` and `--dest-dir`
- copies output files into the test output dir

The generated `test.py` in this checkout points to
`/home/dmedvid/work/pdf2htmlEX/pdf2htmlEX/build/pdf2htmlEX`.

## Non-Browser Tests

`pdf2htmlEX/test/test_output.py` contains Python `unittest` tests that verify:

- default output HTML filenames
- specified output HTML filenames
- split-page output filenames
- `%d` page filename templates
- invalid percent sequences in page filename templates
- issue fixture `issue501`

It does not compare full HTML content.

## Browser Tests

`pdf2htmlEX/test/browser_tests.py` defines `BrowserTests`, which:

- generates output HTML or copies pre-generated HTML
- supports reference-generation mode through `P2H_TEST_GEN`
- screenshots output and reference HTML
- compares screenshots with Pillow `ImageChops.difference`
- writes `.out.png`, `.ref.png`, and `.diff.png`
- copies HTML outputs and diffs to `/tmp/pdf2htmlEX/html`

`test_local_browser.py` creates a Firefox WebDriver, sets the window size, loads
`file://` HTML, waits for `#page-container`, and saves screenshots.

`test_remote_browser.py` defines a Sauce Labs browser matrix. It appears older
than the local browser path and contains Python 2-era `sys.exc_clear()` usage.

## Test Utility Scripts

`pdf2htmlEX/test/compareTestImages`
: Python 3 script that opens output, reference, and diff PNGs from
`/tmp/pdf2htmlEX/png` using the `display` command.

`pdf2htmlEX/test/regenerateTest`
: Python 3 script that copies a generated HTML file from `/tmp/pdf2htmlEX/out`
into the corresponding `browser_tests/<name>/` reference directory. It refuses
to regenerate `test_fail`.

## Historical Packaging Scripts

`archive/build_for_ppa.py`
: Python 2 script for building source packages for old Ubuntu distributions.
It reads version data, updates Debian changelog, creates archives, builds with
`debuild`, and uploads with `dput`.

`archive/createDebianPackage`
: Python 2 script for binary Debian package creation. It is marked with notes
about future improvements and older package workflow assumptions.

These scripts are under `archive/` and should be treated as historical unless a
maintainer explicitly revives them.

## Python Dependencies

Visible test dependencies:

- Python 3 for current tests
- Pillow for browser screenshot diffs
- Selenium for browser automation
- `sauceclient` for remote browser tests

During documentation verification:

- Python `3.14.4` was available
- Pillow was available
- Selenium was not available
