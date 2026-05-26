# HTML Generation

[Documentation Home](../README.md)

The final HTML document is assembled by `HTMLRenderer::post_process` using
`pdf2htmlEX/share/manifest`.

The project wiki frames the generated viewer as content output plus an optional
default UI. `pdf2htmlEX.js` demonstrates how to interact with the generated page
structure, but publishers can replace or wrap that UI with their own code.

## Manifest Structure

`share/manifest` is a small template language:

- lines between `"""` markers are copied literally
- `@filename` embeds or links a file from `data-dir`
- `$css` inserts generated PDF CSS
- `$outline` inserts generated outline HTML
- `$pages` inserts generated page HTML
- comments and empty lines are ignored

The manifest creates:

- HTML doctype and metadata
- static CSS
- generated CSS
- compatibility JavaScript
- `pdf2htmlEX` viewer JavaScript
- sidebar and outline container
- `page-container`
- loading indicator

## Page Structure

Each page has a frame:

```html
<div id="pf<page>" class="pf w<id> h<id>" data-page-no="<page>">
  <div class="pc pc<page> w<id> h<id>">
    ...
  </div>
  <div class="pi" data-data='{"ctm":[...]}'>
  </div>
</div>
```

The page content box contains background assets first, then text, forms, and
link overlays.

The `data-data` JSON currently includes the default CTM for JavaScript.

## Text Lines

`HTMLTextLine::dump_text` emits absolute-positioned text line divs:

```html
<div class="t m... x... h... y... ff... fs... fc... sc... ls... ws...">
  text and spans
</div>
```

Within a line, spans are nested only when state changes. Whitespace offsets are
represented as spans with class `_` and generated width or negative
`margin-left` CSS.

If covered-text detection marks a character covered, `dump_chars` wraps it in a
span with transparent fill and stroke classes. The visual glyph is expected to
come from the background layer.

## Clipping

`HTMLTextPage::clip` records clip state transitions. During text dump, a clip
range can wrap lines in:

```html
<div class="c x... y... w... h...">
  ...
</div>
```

Text lines inside the clip adjust their local left/bottom offsets by the clip
origin.

## Background Layers

Splash PNG/JPEG output emits:

```html
<img class="bi x... y... w... h..." alt="" src="..."/>
```

Cairo SVG output emits either:

```html
<img class="bf" alt="" src="..."/>
```

or:

```html
<embed class="bf" alt="" src="..."/>
```

depending on whether external bitmap references are needed.

## Links

`processLink` emits an anchor when the destination string is not empty, then a
positioned `d` rectangle inside it. Internal PDF destinations are encoded as
fragment links such as `#pf<page>`, with optional `data-dest-detail` consumed by
the viewer JavaScript.

URI links write the URI directly into `href` after HTML attribute escaping.

Unimplemented action types are logged rather than emitted as working links.

## Outlines

`process_outline_items` writes nested `<ul>` and `<li>` structures into the
outline stream. Each item is an anchor with class `l`, an `href`, optional
`data-dest-detail`, and escaped Unicode title text.

The manifest places outline HTML under:

```html
<div id="sidebar">
  <div id="outline">
    ...
  </div>
</div>
```

## Forms

When `--process-form 1`, `process_form` emits:

- `<input type="text">` for Poppler `formText`
- `<div class="ir">` for Poppler `formButton`

Unsupported widget types print a warning. The current form output is basic and
uses inline absolute positioning styles.

## Split Pages

When `--split-pages 1`, each page content is written to a separate page file.
The main pages stream contains empty page frames with `data-page-url`, allowing
the JavaScript viewer to load pages on demand.

`--page-filename` controls the page filename template. If omitted, it is derived
from the input filename as `<input>%d.page`.

## Escaping

`util/encoding.cc` provides:

- `writeUnicodes` for text nodes
- `writeAttribute` for HTML attributes
- `writeJSON` for JSON-like strings

Text and attributes are escaped for common HTML-sensitive characters.
