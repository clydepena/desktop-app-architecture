# Assets

Which assets localization uses, and at which layer.

How source files are imported, cached and cooked into the runtime formats below is described in the
[asset pipeline](../../tool-framework/asset-pipeline/README.md).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Locale** | **supported locales** | which locales the product supports, their fallback chains and display names |
| **Localized content** | **string tables** | keys and text per locale: the source table and the translations |
| | **localized audio** | voice-over per language |
| | **subtitles** | timed text per language |
| | **localized textures and videos** | versions with text or culturally adapted content |
| | **lip-sync data** | per language, matching the recordings |
| **Formatting** | **locale data** | CLDR tables (bundled, or from the platform) |
| **Text rendering** | **fonts per script** | font files chosen for coverage and readability |
| | **font fallback chains** | which fonts to try, in order, per locale |
| **Pipeline** | **translator context** | comments, length limits, speakers, screenshots |
| | **glossaries and translation memory** | consistent terms and reusable past translations |
| | **pseudo-localization settings** | expansion, accents, brackets, right-to-left variants |

## Source and translations

The **source** string table is authored by the team and changes with the product. **Translations** are produced from it
by the pipeline and imported with validation ([The translation pipeline](08-translation-pipeline.md)); they are never
edited in a way that bypasses that validation.

## Settings and runtime state

- **User settings, not assets:** language, regional formats, subtitle and voice language
  ([Settings](../04-settings.md)).
- **Runtime state:** the current locale, caches of resolved strings, dynamic glyph atlases. Never saved.

## Packaging

Locale content is packaged so that it can be installed or downloaded per language: string tables are small and usually
all shipped; voice-over and localized videos are large and often separate packs ([Packages](../io/03-packages.md)).
