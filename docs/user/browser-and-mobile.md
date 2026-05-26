# Browser And Mobile Notes

[Documentation Home](../README.md)

This page summarizes the project wiki's browser and mobile guidance and relates
it to files in this checkout.

## Browser Feature Requirements

Generated output expects basic HTML5 and CSS support. The wiki says advanced
HTML5 APIs such as `<canvas>` and `<video>` are not required for normal output.

Important CSS features:

- `@font-face`
- TrueType or web-font support, depending on `--font-format`
- data URI support when assets are embedded
- CSS transforms
- absolute positioning

JavaScript is only required for the optional default viewer behavior from
`pdf2htmlEX.min.js`. The generated page content and static layout are designed
to remain visible without JavaScript.

The wiki says recent Firefox, recent Chrome, and Internet Explorer 9 or newer
should work. That browser list comes from the wiki and was not revalidated
during this documentation pass.

## Windows Rendering Notes

The wiki suggests enabling ClearType on Windows and mentions MacType as a
possible helper for Chrome font rendering. These are user-environment rendering
tweaks, not converter settings.

## Viewer Assets In This Checkout

The default output shell comes from `pdf2htmlEX/share/manifest`.

The base layout CSS is generated from:

```text
pdf2htmlEX/share/base.css.in
```

The optional visual theme is generated from:

```text
pdf2htmlEX/share/fancy.css.in
```

The default viewer script is generated from:

```text
pdf2htmlEX/share/pdf2htmlEX.js.in
```

`base.css.in` defines `#page-container`, page boxes, background image classes,
text-line classes, clipping boxes, selection behavior, and print styles.

## Mobile Devices

The wiki says mobile devices are not officially supported. Reasons given:

- too many device/browser combinations to test
- mobile browsers may be slow on complicated absolutely positioned pages
- native PDF viewers can be more optimized than mobile browsers for this kind
  of document

The wiki lists user-reported ideas that may improve some mobile cases but may
break others:

- add `-webkit-overflow-scrolling: touch;` to `#page-container` in
  `base.css.in`
- remove off-screen pages from the DOM with JavaScript instead of merely hiding
  them
- trigger GPU acceleration with CSS such as `translateZ` or `translate3d`

These are not implemented as default behavior in this checkout. Apply them as
site-specific viewer experiments and test with representative files.

## Practical Compatibility Checklist

When output looks wrong in a browser:

1. Confirm the browser supports web fonts, data URIs, transforms, and absolute
   positioning.
2. Try non-embedded output so CSS, fonts, and background files can be inspected
   separately.
3. Confirm font files are served with usable MIME types by the hosting server.
4. Check whether JavaScript is blocked; this affects the default viewer UI and
   split-page lazy loading.
5. Test with `--fallback 1` to determine whether the issue is in native text
   rendering or in background rendering.
