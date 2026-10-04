# Localized assets

Localization is not only strings. Many assets have versions per locale.

## What gets localized

- **Voice-over:** dialogue recorded per language. Dialogue events pick the line for the current locale
  ([Sound events](../audio/06-sound-events.md)), falling back to another language with subtitles when a voice isn't
  available.
- **Subtitles:** timed to the localized audio, not the original.
- **Lip-sync data** matching each language's recording.
- **Textures with text in them:** signs, logos, UI images. Better still: avoid baking text into images, and draw it as
  localized text over a text-free image.
- **Videos and cutscenes**, or their subtitle tracks.
- **Fonts** per script ([Text rendering and layout](05-text-rendering-and-layout.md)).

## Cultural adaptation

Some content changes meaning between regions, not just language:

- symbols, gestures and colors that mean different things
- maps and borders, which can be politically sensitive
- content restricted or rated differently by region (age ratings, legal requirements)
- names, holidays and examples that make sense locally

These are product decisions, made per region, delivered as assets in the same way as translations.

## Overlays per locale

Localized assets are delivered through the [virtual file system](../io/02-virtual-file-system.md): a folder or archive
per locale is mounted **above** the base content.

```
content:  ->  1. loc/de/          (German versions, where they exist)
              2. content.pak      (the default versions)
```

`content:/ui/logo.png` resolves to the German logo when one exists, and to the default otherwise. Code always asks for
the same path. This is the same overlay mechanism as patches and mods, with the same idea as the string
**fallback chain** ([Locales](01-locales.md)).

## Downloads and sizes

Voice-over is large. Applications often ship the user's language and download others on demand, so locale packs are
packaged as separate archives or chunks ([Packages](../io/03-packages.md)), and switching to a language whose voice pack
isn't installed may need a download first.
