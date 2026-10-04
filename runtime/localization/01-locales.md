# Locales

A **locale** identifies a language and, optionally, a region and a script. It is written as a standard tag (BCP 47):

| Tag | Meaning |
|---|---|
| `en-US`, `en-GB` | English, with US or UK conventions |
| `pt-BR`, `pt-PT` | Brazilian or European Portuguese |
| `zh-Hans`, `zh-Hant` | Chinese in simplified or traditional script |
| `sr-Latn`, `sr-Cyrl` | Serbian in Latin or Cyrillic script |
| `es-419` | Spanish for Latin America |

## Language, region and script are different things

- **Language:** which words to use.
- **Region:** conventions such as number and date formats, currency, measurement units, the first day of the week.
- **Script:** which writing system (Latin, Cyrillic, Han in simplified or traditional form).

A Swiss user may read German but expect Swiss number formats; an English speaker living in Germany may want English
text with German dates. Operating systems let users choose language and regional formats separately, and applications
can follow them.

## The fallback chain

Translations are rarely complete everywhere. When a string is missing in the current locale, the next one in a
**fallback chain** is tried:

```
pt-BR  ->  pt  ->  en (the source language)
```

So a partial translation shows mostly Portuguese with a few English strings, never blanks or raw keys. This is the
**Chain of Responsibility** pattern, and the same chain applies to localized assets
([Localized assets](06-localized-assets.md)).

## Locale data

Plural rules, number formats, month and day names, sorting rules, currency symbols and much more come from the
**Unicode CLDR** (Common Locale Data Repository), usually through a library (**ICU**, or the platform's own locale
APIs). Nobody should write these tables by hand: they are large, subtle, and updated regularly.

## Flags aren't languages

A flag is a country. Many countries share languages (Spanish across two continents) and many countries have several
(Switzerland, India, Canada). Language pickers show **language names, written in that language**: "Deutsch",
"日本語", "Español", "Português (Brasil)", so people can find their language without reading the current one.

## Supported locales

An application declares which locales it supports, each with its fallback chain and display name
([Assets](12-assets.md)). The user's preferred languages (from the OS) are matched against that list
([Choosing and switching the locale](07-choosing-and-switching.md)).
