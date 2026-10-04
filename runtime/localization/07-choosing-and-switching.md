# Choosing and switching the locale

## The default

The operating system gives an ordered list of the user's preferred languages. The application **matches** it against
the locales it supports, with fallback: an OS set to `es-MX` gets the application's `es-419` or `es` if there is no
`es-MX`; an OS set to an unsupported language gets the next language in the user's list, then the source language.

## The user's override

A **language setting** in the application overrides the OS choice ([Settings](../04-settings.md)). Some applications
also offer separate settings for **regional formats**, **subtitle language** and **voice language** (people often want
original voices with subtitles in their own language).

## Switching at runtime

Changing the language without restarting is easy when everything is a key, and nearly impossible when text was copied
into widgets at creation time:

```
locale changed (event)
  -> UI re-resolves every key and re-lays out (text lengths and direction change)
  -> localized assets reload through their handles (overlays now point elsewhere)
  -> fonts and glyph atlases switch if the script changed
```

This is the **Observer** pattern. Things that genuinely need a restart or a download (a voice pack not installed) say
so explicitly.

## Multiplayer and servers

Each player has their own locale. So:

- the server sends **keys and arguments** ("item.pickedUp", item = 42), never translated text
- each client formats the message in its own language
- **chat** is user-written text and is not translated by the localization system (automatic translation is a separate
  feature, if offered at all)
- server-side logic never depends on any player's language

## Logs and errors

- **Log messages, internal errors and crash reports stay in one language.** Developers must be able to read them,
  search them, and match them across machines.
- **User-facing error messages are localized**, and include a stable **error code**, so support can find the matching
  log entry whatever language the user saw.
