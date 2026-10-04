# Text rendering and layout

Turning localized text into pixels is mostly done by the UI toolkit and text renderer, but localization decides what
they must support. Each item below is a common source of bugs when only English was tested.

## Unicode and grapheme clusters

- **Unicode everywhere**, with UTF-8 as the usual storage and interchange format.
- A visible character (a **grapheme cluster**) can be several code points: a letter with combining accents, an emoji
  with a skin tone, a family emoji, a flag.
- Truncating, reversing or moving a cursor by **bytes or code points** breaks characters apart. Text operations
  (cursor movement, deletion, truncation, ellipsis) work on grapheme clusters.

## Shaping

In Arabic, letters change shape depending on their neighbors (initial, middle, final, isolated forms); Indic scripts
combine and reorder characters; many fonts use ligatures. A **text shaper** (HarfBuzz is the most common) turns a string
and a font into positioned glyphs. Drawing "one glyph per character" is wrong for these scripts.

## Bidirectional text

Arabic and Hebrew run right to left, but numbers and embedded Latin words inside them run left to right. The **Unicode
Bidirectional Algorithm** decides the visual order of mixed text. Text editing (cursor movement, selection) in mixed
directions follows it too.

## Mirrored layouts

In right-to-left languages, the whole interface is mirrored: menus open from the right, "back" arrows point right,
progress bars fill from the right, the order of buttons flips. Layouts are written with **start and end** instead of
left and right, so they flip automatically. Some things don't mirror (media playback controls, clocks, images of
real objects).

## Line breaking

- Chinese and Japanese can break between almost any two characters, with exceptions for punctuation.
- Thai, Lao, Khmer and Burmese write words without spaces; finding break points needs a **dictionary**.
- Some punctuation must never start or end a line.

The line breaker follows the Unicode line-breaking rules per language, rather than "break at spaces".

## Fonts and fallback

- No single font covers every script. A **fallback chain** per locale picks the next font when a glyph is missing, and
  the order matters (Japanese and Chinese share characters but prefer different glyph shapes).
- CJK fonts contain tens of thousands of glyphs, so glyph atlases are built **dynamically** from the characters actually
  shown, and their memory is a budgeted category ([Memory](../memory/README.md)).
- Fonts are chosen per locale for readability, not only for coverage.

## Text expansion

Translations are often 30 to 40% longer than English, and short labels can double ("OK" is short almost everywhere;
"Settings" is not). Layouts must:

- **grow** (boxes sized by their content), or
- **wrap** onto more lines, or
- **shrink** the text within limits,

and **truncate with an ellipsis** only as a last resort, with the full text available on hover. Fixed-width boxes
sized for English are the most common localization bug.

## Vertical text

Traditional Japanese and Chinese can be written in vertical columns. It is rare in applications, but some content
(books, signs in a game world) needs it.
