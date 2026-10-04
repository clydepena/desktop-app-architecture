# Coupling: what may depend on localization

The direction rule: consumers depend on the localization service's interface; the service depends only on the runtime
below it (IO for tables, resources for localized assets, settings for the user's choice, platform locale data). It
never knows where its text is displayed.

## Safe to couple

| System | Why it's safe |
|---|---|
| **UI toolkit, text rendering** | resolve keys, format values, use text services |
| **Audio, rendering** | pick localized assets through overlays and handles |
| **Data description, inspector** | the localized-text field type |
| **Translation tools, string editor** | the tool layer, above localization |
| **Settings** | the user's language and format choices |

## Keep decoupled

| System | How it handles text instead |
|---|---|
| **Domain logic** | ids and keys only; never compares, parses or branches on translated text |
| **Save files, network messages, logs read by tools** | keys, ids and locale-independent formats; never translated or locale-formatted text |
| **Developer diagnostics** | one fixed language; error codes link user-facing messages to log entries |
| **Input bindings** | bound to physical keys and actions; only their display names are localized |

## Mistakes to avoid

- **Hard-coded user-visible strings**, and sentences built by concatenation.
- **`n == 1` plurals**, and positional placeholders.
- **One key reused for words** that are the same in English but not in other languages.
- **Locale-formatted numbers or dates in files and messages** (the decimal comma bug).
- **Splitting or truncating text by bytes or code points.**
- **Fixed-width layouts sized for English.**
- **Text baked into images.**
- **Assuming left to right**, or assuming one font covers every script.
- **Locale-dependent upper and lower case in comparisons** (the Turkish "i").
- **Flags as language pickers.**
- **Translated text sent over the network or stored in saves.**
- **Localized log messages** that developers need to read.

## Testing the boundary

- **Pseudo-localization** in everyday development builds: brackets, accents and expansion, plus a right-to-left
  pseudo-locale ([The translation pipeline](08-translation-pipeline.md)).
- **Automated checks** on every build: missing keys, placeholder mismatches, incomplete plural forms, strings over their
  limits, unused keys.
- **Screenshot runs** in the longest languages and in right to left, compared for overflow and clipping.
- **Running the test suite under unusual system locales** (German, Turkish, Arabic) to catch formatting and casing bugs
  in code that should be locale-independent.
