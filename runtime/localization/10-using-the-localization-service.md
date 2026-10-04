# Using the localization service

How the rest of an application gets text and formats values: through keys and formatting calls, resolved at the moment
they are shown, re-resolved when the locale changes.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Text** | resolve keys to text | `text(key)`, `text(key, args)` with placeholders, plurals and selects |
| **Formatting** | numbers, dates, currencies, units | `formatNumber(v)`, `formatDate(d, style)`, `formatCurrency(v, code)`, `formatDuration(t)` |
| **Locale** | current and supported locales | `current()`, `setLocale(tag)`, `fallbackChain()`, `supportedLocales()`, `direction()` |
| **Events** | react to a change | `onLocaleChanged(handler)` |
| **Assets** | localized versions | resolved automatically through locale overlays; `localizedPath(path)` when needed |
| **Collation** | sort and compare for display | `compare(a, b)`, `sort(list)` by the current locale |
| **Text services** | used by the UI toolkit | segmenting into graphemes, words and lines; bidirectional ordering; font fallback per script |
| **Debug** | development | `enablePseudoLocale(kind)`, missing-key reporting, show-keys mode |

## How each part of an application uses it

| Part | How it uses localization |
|---|---|
| **UI toolkit** | every label is a key, resolved and re-resolved on locale change; layouts use start and end and mirror for right to left; boxes grow or wrap; text goes through shaping, bidirectional ordering, line breaking and font fallback |
| **Domain / gameplay** | stores keys and ids (item names, quest text, dialogue lines), never translated text; never branches on displayed text |
| **Data description** | the localized-text field type, so extraction, the inspector and validation know which fields hold keys ([Data description](../08-data-description.md)) |
| **Audio** | dialogue events pick the voice line for the current locale, with subtitle fallback; subtitles timed to the localized audio |
| **Rendering** | glyph atlases built dynamically per script; fonts per locale; localized textures through overlays |
| **Input** | text entry through input methods ([Text input](../input/06-text-input.md)); button prompts and key names localized, while bindings stay on physical keys |
| **IO** | locale overlays in the virtual file system; string tables loaded per locale |
| **Persistence and saves** | keys, ids and locale-independent numbers and dates; never translated text |
| **Networking** | sends keys and arguments; each client formats them |
| **Scripting** | scripts use keys and the formatting API; string literals in scripts are flagged |
| **Settings** | language, regional formats, subtitle and voice language |
| **Diagnostics** | logs and internal errors in one language; user-facing errors localized, with error codes |
| **Memory** | CJK fonts and glyph atlases are large, budgeted categories |
