# The translation pipeline

The workflow that keeps translations in step with a product that keeps changing.

## The steps

1. **Extraction:** tools collect every key and source string from code, data and assets. Localized fields in the data
   description ([Strings and keys](02-strings-and-keys.md)) make this reliable: no searching for strings by guesswork.
2. **Export** to translators in a standard format: XLIFF or PO for translation tools, spreadsheets or JSON for smaller
   teams. Each string carries its **context**: comments, maximum length, speaker, screenshots.
3. **Translation**, often in a translation management system with:
   - a **translation memory**, which reuses previous translations of the same or similar sentences
   - a **glossary**, which keeps important terms consistent ("Health" is always the same word)
4. **Import with validation**, rejecting or flagging:
   - placeholders that don't match the source
   - missing plural forms for that language
   - text over its length limit
   - unbalanced markup
   - missing or unknown keys
5. **Review in context:** translations checked in the running application, not only in a spreadsheet.

## Change tracking

When a source string changes, its translations are marked **stale**: shown with the old translation (or the source
text) and queued for review. They are never silently kept as if still correct, nor silently discarded.

New strings fall back through the locale chain until translated, so development is never blocked by missing
translations.

## String freeze

Before a release, source text stops changing (a **string freeze**), so translators can finish and review everything.
Late text changes after the freeze are tracked as exceptions.

## Pseudo-localization

The most valuable early tool. Replace every string with an accented, lengthened, bracketed version:

```
"New Game"  ->  "[Ñéŵ Ĝåḿé ~~~~]"
```

Running the application in this pseudo-locale from day one reveals, long before real translations exist:

| What you see | What it means |
|---|---|
| text without brackets | hard-coded text that bypassed the string tables |
| a missing closing bracket | truncation: the box is too small |
| boxes instead of accented letters | a font missing glyphs |
| broken layouts | boxes that can't grow by 30-40% |

A **right-to-left pseudo-locale** (text reversed and marked as right to left) does the same for mirroring and
bidirectional text. Pseudo-localization is a text provider wrapping the real one: the **Decorator** pattern.

## Continuous localization

Instead of translating everything at the end, many teams send new strings to translators continuously, as they are
added, and import translations back automatically with validation. Small batches keep translations close to complete at
all times.
