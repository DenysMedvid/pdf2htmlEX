# Wiki Source Notes

[Documentation Home](../README.md)

This page records the project wiki material read during the local documentation
update. The source was cloned from:

```text
https://github.com/pdf2htmlEX/pdf2htmlEX.wiki.git
```

The wiki is useful project documentation, but some pages are explicitly old or
version-specific. Local docs therefore paraphrase the wiki and cross-check
against this checkout when possible instead of treating every statement as
current implementation truth.

## Localized Into These Docs

| Wiki Page | Local Destination |
| --- | --- |
| `Building.md` | [Build Overview](../build/overview.md), [Build From Source](../build/build-from-source.md), [Platforms](../build/platforms.md), [Prebuilt Packages](../build/prebuilt-packages.md) |
| `Quick-Start.md` | [Quick Start](../user/quick-start.md) |
| `FAQ.md` | [FAQ And Limitations](../user/faq-limitations.md), [Troubleshooting](../build/troubleshooting.md) |
| `Feature-List.md` | [FAQ And Limitations](../user/faq-limitations.md), [Architecture Overview](../architecture/overview.md) |
| `Limitations.md` | [FAQ And Limitations](../user/faq-limitations.md), [Debugging](../development/debugging.md) |
| `Unfeatures.md` | [FAQ And Limitations](../user/faq-limitations.md) |
| `Reflowable-Text.md` | [FAQ And Limitations](../user/faq-limitations.md) |
| `Customizing-Output.md` | [Optimization And Customization](../user/optimization-and-customization.md), [Asset Output](../internals/asset-output.md) |
| `Optimizing-Output.md` | [Optimization And Customization](../user/optimization-and-customization.md) |
| `Optimizing-PDF-Files.md` | [Optimization And Customization](../user/optimization-and-customization.md) |
| `Font-Files.md` | [Font Handling](../internals/font-handling.md), [Optimization And Customization](../user/optimization-and-customization.md) |
| `Browser-Requirements.md` | [Browser And Mobile Notes](../user/browser-and-mobile.md) |
| `Mobile-Devices.md` | [Browser And Mobile Notes](../user/browser-and-mobile.md) |
| `Troubleshooting-Crashes.md` | [Troubleshooting](../build/troubleshooting.md), [Debugging](../development/debugging.md) |
| `Download-*.md` | [Prebuilt Packages](../build/prebuilt-packages.md), [Platforms](../build/platforms.md) |
| `Auto-Tests.md` | [Testing](../development/testing.md) |
| `Building-Continuous-Integration.md` | [Testing](../development/testing.md), [Platforms](../build/platforms.md) |
| `_Footer.md` | [Optimization And Customization](../user/optimization-and-customization.md) |

## Page Inventory

| Page | Notes |
| --- | --- |
| `Home.md` | Wiki navigation page. It links introduction, features, limitations, downloads, build, quick start, browser requirements, command-line options, customization, optimization, fonts, mobile notes, and the TUG 2013 talk. |
| `Introduction.md` | Explains the project goal: convert PDF to HTML for plugin-free web publishing while preserving visual fidelity and selectable text when possible. Also notes that PDF-to-HTML is not lossless. |
| `Feature-List.md` | Lists native/selectable text, font/color/position preservation, flexible output modes, links, outlines, printing, clipping, SVG backgrounds, and Type 3 font support. |
| `Limitations.md` | Lists hard cases: reflowable text, stroked text, clipped/overlapped text, writing-mode fonts, and browser font-size rounding. |
| `Unfeatures.md` | States that general PDF manipulation, PDF optimization, and full viewer UI features are outside the expected converter scope. |
| `Reflowable-Text.md` | Explains why reconstructing reflowable paragraphs from fixed-layout PDF lines is difficult. |
| `Building.md` | Current-looking build overview. Emphasizes scripted builds, pinned Poppler/FontForge source versions, Debian/Alpine package-manager paths, and static linking. |
| `Building-Continuous-Integration.md` | Notes Travis CI on Ubuntu Bionic and local Laminar CI across Ubuntu and Alpine Docker images. Local repository now also has GitHub Actions on Ubuntu 22.04. |
| `Building-Old-Information.md` | Explicitly old. Mentions Homebrew, MSYS2/mingw-w64, old distro package dependencies, Poppler xpdf headers, libspiro, Python 2 header issues, and old manual CMake builds. |
| `Download-Debian-Archive.md` | Describes installing release `.deb` files with `apt install ./file.deb`; notes host Fontconfig/iconv configuration still matters. |
| `Download-Alpine-Tar-Archive.md` | Describes Alpine tar archive plus installer script; warns about Alpine `iconv` and fonts. |
| `Download-AppImage.md` | Describes AppImage usage, WSL/Ubuntu compatibility notes, macOS limitation, and host configuration dependencies. |
| `Download-Docker-Image.md` | Gives Docker run/alias examples and container mount points for `/pdf`, data dir, poppler-data, and `/etc/fonts`. |
| `Download-Old-Information.md` | Explicitly old. Lists historical third-party packages/images and a deprecated Ubuntu PPA. |
| `Download.md` | Empty in the cloned wiki checkout. |
| `Quick-Start.md` | Common recipes for zoom, page ranges, JPEG backgrounds, separate publisher assets, split pages, and fallback mode. |
| `Command-Line-Options.md` | Manpage for version `0.12`; useful historical option descriptions, but current source/help/manpage supersede it. |
| `FAQ.md` | Common user issues: manifest path, browser requirements, output size, text readability, ToUnicode/copy-paste, missing images, blur. |
| `Font-Files.md` | Explains `pdffonts`, embedded versus external fonts, standard PDF fonts, browser font availability, default external-font embedding, and duplicate font output. |
| `Customizing-Output.md` | Explains data-dir, manifest, base/fancy CSS, viewer JS, custom CSS after `$css`, and experimental manifest syntax. |
| `Optimizing-Output.md` | Discusses static resource caching, embedding tradeoffs, HTTP compression, PNG optimization, and PDF pre-optimization. |
| `Optimizing-PDF-Files.md` | Suggests optimizing PDFs before conversion and mentions Ghostscript. |
| `Browser-Requirements.md` | Lists required browser features: basic HTML5, web fonts, data URIs, transforms, absolute positioning, and optional JavaScript. |
| `Mobile-Devices.md` | Says mobile is unsupported and lists experimental performance ideas. |
| `Auto-Tests.md` | Mentions Travis testing and points to `pdf2htmlEX/test/README.md`. |
| `Auto-Tests-Old-Information.md` | Explicitly old. Mentions Travis plus Sauce Labs matrix testing. |
| `Troubleshooting-Crashes.md` | Gives a crash workflow centered on `pdf2htmlEX -v`, `--debug 1`, kept temp files, and FontForge isolation. |
| `Related-Projects.md` | Lists `fmalina/transcript` as PDF-to-semantic-HTML related work. |
| `How-can-I-help.md` | Historical community/contribution page. Contains links to the original project and contribution guidance. |
| `How-can-I-help?.md` | Empty in the cloned wiki checkout. |
| `Try-Your-Own-Files.md` | Contains only informal placeholder text in the cloned wiki checkout. |
| `Original-Author's-Words.md` | Historical origin note from Lu Wang dated 2013-02-01. |
| `_Footer.md` | Contains an unverified WebP conversion/base64 embedding shell workflow. |

## Important Version Caveats

- `Command-Line-Options.md` says it is the manual page for version `0.12`.
  Current options are documented from `pdf2htmlEX/src/pdf2htmlEX.cc`,
  `pdf2htmlEX/pdf2htmlEX.1.in`, and `pdf2htmlEX/build/pdf2htmlEX --help`.
- Old wiki pages intentionally keep historical package names and dependency
  versions. They are not used as current build instructions.
- The wiki says `--hdpi`/`--vdpi` in older command-line docs. Current source
  registers `--dpi`.
- The wiki mentions Travis and Laminar CI. This checkout also has
  `.github/workflows/build.yml`.
- The wiki's third-party package lists are historical leads, not verified
  package support for this checkout.
