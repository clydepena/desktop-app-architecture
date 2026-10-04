# Localization

How an application becomes usable in other languages and regions: the words, but also numbers, dates, plurals, text
direction, fonts, voices and images.

Two terms are often mixed up:

- **Internationalization ("i18n")** is *engineering* the application so it **can** be localized: no hard-coded text,
  flexible layouts, Unicode everywhere, formatting by locale. Done once, by programmers.
- **Localization ("l10n")** is *doing* it for each language and region: translating, recording voices, adapting images.
  Done per locale, mostly by translators.

It has the familiar shape (a platform layer, an abstraction, data-driven content, a narrow interface, the same
dependency direction). Its special problem is that **every assumption about text is wrong somewhere**: plurals aren't
"one vs many", word order differs, German runs about 30% longer than English, Arabic runs right to left, Thai has no
spaces between words, Chinese needs fonts with tens of thousands of characters, and the uppercase of "i" isn't "I" in
Turkish.

## The layers inside a localization service

```
+---------------------------------------------------------------------+
| Consumers          UI, gameplay text, dialogue and subtitles, item      |  owned by the app
|                    names, store pages, help                             |
+---------------------------------------------------------------------+
| Translation        extracting strings, translator context, import       |  tools and workflow
| pipeline           checks, pseudo-localization, change tracking         |
+---------------------------------------------------------------------+
| Localized content  string tables, localized audio, textures, video,     |  data per locale
|                    subtitles, fonts; overlays per locale                 |
+---------------------------------------------------------------------+
| Formatting         messages with placeholders, plurals, gender,         |  rules per locale
|                    numbers, dates, currencies, units, sorting            |
+---------------------------------------------------------------------+
| Text rendering     Unicode, shaping, bidirectional text, line           |  with the UI toolkit
| and layout         breaking, fonts and fallback, text expansion         |
+---------------------------------------------------------------------+
| Locale             the current language and region, user choice,        |
|                    fallback chain, switching at runtime                  |
+---------------------------------------------------------------------+
| Platform           OS locale settings, locale data (CLDR / ICU),        |  platform layer
|                    installed fonts, input methods                        |
+---------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Locales](01-locales.md) | locale tags, language vs region vs script, fallback chains, locale data, why flags aren't languages |
| 2 | [Strings and keys](02-strings-and-keys.md) | no text in code, string tables, no concatenation, stable keys, context, localized fields |
| 3 | [Message formatting](03-message-formatting.md) | placeholders, plurals in every language, gender and agreement, message format standards |
| 4 | [Numbers, dates and data formats](04-numbers-dates-and-data-formats.md) | formatting for people vs files, units, calendars, collation, case conversion |
| 5 | [Text rendering and layout](05-text-rendering-and-layout.md) | graphemes, shaping, bidirectional text, mirroring, line breaking, fonts, expansion |
| 6 | [Localized assets](06-localized-assets.md) | voice, subtitles, textures, video, cultural adaptation, overlays per locale |
| 7 | [Choosing and switching the locale](07-choosing-and-switching.md) | defaults, overrides, runtime switching, multiplayer, logs |
| 8 | [The translation pipeline](08-translation-pipeline.md) | extraction, context, validation, change tracking, string freeze, pseudo-localization |
| 9 | [Editors and tools](09-editors-and-tools.md) | string table editing, previews, overflow checks, localized fields |
| 10 | [Using the localization service](10-using-the-localization-service.md) | the public interface, and how each part of an app uses it |
| 11 | [Coupling: what may depend on localization](11-coupling.md) | safe and unsafe dependencies, pseudo-locales, unusual system locales |
| 12 | [Assets](12-assets.md) | string tables, localized media, fonts, locale data, translator context |

## How localization compares

| | Input | IO | Scripting | Localization |
|---|---|---|---|---|
| Role | intentions in | storage | behavior from others | **text and content for every language and region** |
| Platform layer | OS input, IME | file systems | runtimes | **OS locale settings, CLDR / ICU data, fonts** |
| Indirection | actions over devices | logical paths over mounts | handles over host objects | **keys over text; overlays over assets** |
| Fallbacks | n/a | overlay search order | n/a | **the locale fallback chain** |
| Test version | injected input | in-memory files | fake host | **pseudo-locales** |

## Patterns in localization

| Pattern | Where |
|---|---|
| Indirection by key | the core idea: text referred to by keys, resolved per locale |
| Chain of Responsibility | the locale fallback chain |
| Strategy | plural rules and formatters per locale |
| Interpreter | message formats (a tiny language for plurals and selects) |
| Decorator | pseudo-localization wrapping the real text provider |
| Observer | locale changed |
| Overlay | localized assets through the virtual file system |
| Flyweight | one shared string table per locale |

## Related

- [Sound events](../audio/06-sound-events.md): dialogue per locale.
- [Text input](../input/06-text-input.md): typing in every language.
- [The virtual file system](../io/02-virtual-file-system.md): locale overlays.
- [UI toolkit](../ui/README.md): widgets, layout, focus and styling on top of windowing, rendering, input and localization.
- [Data description](../08-data-description.md): the localized-text field type.
