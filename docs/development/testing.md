# Testing

[Documentation Home](../README.md)

The repository has a fast non-browser output suite and a slower browser
screenshot suite.

## Test Structure

| Path | Purpose |
| --- | --- |
| `pdf2htmlEX/test/test_output.py` | Non-browser unit tests. They check that conversions succeed and expected output filenames are produced. |
| `pdf2htmlEX/test/test.py.in` | CMake template for common test utilities and paths. |
| `pdf2htmlEX/test/test.py` | Generated local test helper from the current build. |
| `pdf2htmlEX/test/browser_tests.py` | Shared browser screenshot comparison logic. |
| `pdf2htmlEX/test/test_local_browser.py` | Local Firefox/Selenium test runner. |
| `pdf2htmlEX/test/test_remote_browser.py` | Sauce Labs browser matrix runner. |
| `pdf2htmlEX/test/browser_tests/` | Browser-test PDFs and reference HTML. |
| `pdf2htmlEX/test/test_output/` | PDFs and fixtures for non-browser output tests. |
| `pdf2htmlEX/test/runLocalTestsPython` | Runs `python3 test_output.py`. |
| `pdf2htmlEX/test/runLocalBrowserTests` | Starts Xvfb and runs local browser tests. |
| `buildScripts/runTests` | Installs automatic test software, runs shell/basic tests, runs browser tests, zips results. |

`pdf2htmlEX/CMakeLists.txt` registers:

```text
test_basic   python <source>/test/test_output.py
test_browser python <source>/test/test_local_browser.py
```

The wiki's `Auto-Tests.md` page points readers to this test directory. It also
mentions Travis CI on Ubuntu Bionic. Another wiki CI page mentions Laminar-based
local CI across Ubuntu and Alpine Docker images. In this checkout, the visible
current hosted workflow is `.github/workflows/build.yml` on Ubuntu 22.04, while
`.travis.yml` remains as older CI configuration.

## Fast Non-Browser Tests

Run from the test directory:

```sh
mkdir -p /tmp/pdf2htmlEX/tmp /tmp/pdf2htmlEX/dat /tmp/pdf2htmlEX/png /tmp/pdf2htmlEX/out /tmp/pdf2htmlEX/html
cd pdf2htmlEX/test
python3 test_output.py
```

The generated `test.py` in this checkout expects:

- binary: `/home/dmedvid/work/pdf2htmlEX/pdf2htmlEX/build/pdf2htmlEX`
- temp dir: `/tmp/pdf2htmlEX/tmp`
- data dir: `/tmp/pdf2htmlEX/dat`
- output dir: `/tmp/pdf2htmlEX/out`
- PNG dir: `/tmp/pdf2htmlEX/png`
- HTML report dir: `/tmp/pdf2htmlEX/html`

## Browser Tests

Local browser tests require:

- Firefox
- geckodriver
- Xvfb
- Python `selenium`
- Pillow

Install helpers:

```sh
pdf2htmlEX/test/installAutomaticTestSoftwareApt
pdf2htmlEX/test/installAutomaticTestSoftwareDnf
```

Run:

```sh
cd pdf2htmlEX/test
./runLocalBrowserTests
```

The browser suite:

1. generates or copies HTML output for each PDF
2. screenshots generated and reference HTML
3. uses Pillow `ImageChops.difference`
4. writes `.out.png`, `.ref.png`, and `.diff.png`
5. fails if the diff image has a bounding box, except for the intentional
   `test_fail` case

`browser_tests/text_visibility` is skipped in code because of clipping issues.
`pdf2htmlEX/test/README.md` also records this as currently failing.

## Reference Regeneration

`P2H_TEST_GEN=1` enables generation mode. In that mode, browser tests copy
generated HTML into reference folders instead of comparing.

Manual helper:

```sh
cd pdf2htmlEX/test
./regenerateTest <testName>
```

Image comparison helper:

```sh
cd pdf2htmlEX/test
./compareTestImages <testName>
```

It opens output, reference, and diff PNG files from `/tmp/pdf2htmlEX/png`.

## Commands Run During Documentation

Environment/tool check:

```sh
python --version
python3 --version
cmake --version | head -1
make --version | head -1
java -version 2>&1 | head -3
which firefox
which geckodriver
which Xvfb
```

Result:

- Python `3.14.4`
- CMake `4.3.0`
- GNU Make `4.4.1`
- OpenJDK `25.0.3`
- Firefox available
- geckodriver missing
- Xvfb missing

Build verification:

```sh
cmake --build pdf2htmlEX/build --target pdf2htmlEX -- -j2
```

Result: failed during CMake configure check because CMake 4.3.0 rejects the
project's `cmake_minimum_required(VERSION 2.6.0)`.

Existing binary version check:

```sh
pdf2htmlEX/build/pdf2htmlEX -v
```

Result: passed. The binary reported `pdf2htmlEX 0.18.8.rc2`, Poppler `24.06.1`,
FontForge `20230101`, Cairo `1.18.4`, and image formats `png jpg svg`.

Non-browser direct tests:

```sh
mkdir -p /tmp/pdf2htmlEX/tmp /tmp/pdf2htmlEX/dat /tmp/pdf2htmlEX/png /tmp/pdf2htmlEX/out /tmp/pdf2htmlEX/html
cd pdf2htmlEX/test
python3 test_output.py
```

Result: passed, 23 tests.

CTest listing:

```sh
ctest --test-dir pdf2htmlEX/build -N
```

Result: found `test_basic` and `test_browser`.

CTest basic test:

```sh
ctest --test-dir pdf2htmlEX/build -R test_basic --output-on-failure
```

Result: passed, 1 test.

Browser dependency check:

```sh
python3 - <<'PY'
for mod in ['selenium','PIL']:
    try:
        __import__(mod)
        print(f'{mod}: available')
    except Exception as e:
        print(f'{mod}: unavailable ({e.__class__.__name__}: {e})')
PY
```

Result: `PIL` available, `selenium` unavailable. Browser tests were not run
because Selenium, geckodriver, and Xvfb were missing.

## Coverage Limitations Visible From Tests

The non-browser tests validate file creation and selected output names. They do
not deeply inspect generated HTML/CSS content.

The browser tests provide visual coverage for selected PDFs, but they require a
specific browser/test environment and currently skip `text_visibility`.

There is no visible unit-test layer for individual C++ classes such as
`StateManager`, `DrawingTracer`, or `ffw`.
