# Numbers, dates and data formats

The same values are written differently by people in different regions:

| Value | en-US | de-DE | fr-FR |
|---|---|---|---|
| 1234.5 | 1,234.5 | 1.234,5 | 1 234,5 |
| a date | 10/4/2026 | 4.10.2026 | 04/10/2026 |
| a currency amount | $1,234.50 | 1.234,50 € | 1 234,50 € |
| a percentage | 50% | 50 % | 50 % |

## Format for people, never for files

- Values **shown to people** are formatted with the user's locale.
- Values **saved, sent or parsed by programs** (save files, settings, network messages, logs read by tools) use a
  fixed, **locale-independent** format.

A classic bug: a computer set to German writes `1,5` into a save file through a locale-aware formatter; the parser
expecting `1.5` then reads 1, 15 or fails. The fix is never "parse both"; it is to format data files without the locale
at all. Many languages' standard functions use the system locale by default, so this has to be deliberate.

## Units, time and calendars

- **Measurement units:** metric or imperial; temperature in Celsius or Fahrenheit.
- **Time zones:** store times in UTC (or with an explicit zone), convert for display.
- **12 or 24 hours**, and the **first day of the week** (Sunday or Monday).
- **Calendars:** not every locale uses the Gregorian calendar by default; locale libraries handle the others.
- **Relative times:** "3 days ago", "in 2 hours", with plural rules again.

## Sorting

Sorting text for display follows the locale's **collation** rules:

- in Swedish, "ö" sorts after "z"; in German, next to "o"
- some languages treat letter pairs as single letters for sorting
- accents and case may matter, or not, depending on the locale

Internal sorting (keys, ids, file names in an archive) uses a fixed byte order, so it is stable on every machine.

## Case conversion depends on the locale

- Turkish uppercases "i" to "İ" (dotted) and lowercases "I" to "ı" (dotless).
- German "ß" uppercases to "SS".
- Greek has a final form of sigma.

Upper-casing strings to compare them breaks in some locales, typically by making a keyword or file name stop matching
when the system locale is Turkish. Compare **ids**, or use explicit locale-independent or locale-aware comparison
functions, never "uppercase both and compare".
