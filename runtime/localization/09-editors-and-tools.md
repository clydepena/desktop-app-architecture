# Editors and tools

## A string table editor

String tables are documents ([Documents](../../tool-framework/02-documents.md)), edited with undo like any other:

- search by key, source text or translation
- the source and a translation side by side
- markers for **stale**, **missing** and **over-length** entries
- "where is this key used?", across code, data and assets
- context fields (comments, length limits, speaker, screenshot) edited next to the text

## Preview in any locale

Editors let a developer view the UI in any supported locale, a **pseudo-locale** or **right to left**, without changing
the operating system's language. A "show keys" mode displays keys instead of text, to find which key a label uses.

## Overflow checks

Text that doesn't fit its box is flagged:

- live in the editor, as you look at a screen in a long language
- in **automated screenshot runs** that visit screens in every locale and report clipping and overflow

## Localized fields in the inspector

Fields of the localized-text type ([Strings and keys](02-strings-and-keys.md)) show a **key picker** with a preview of
the text in the current editing locale, instead of a free text box. Typing raw text into a localized field is a
validation warning; the editor offers to create a new key for it instead.

## The editor's own UI

An editor's own menus and panels are localized too, but separately from the product's content: different strings,
different translators, different release cycles. Mixing the two string tables makes both harder to manage.
