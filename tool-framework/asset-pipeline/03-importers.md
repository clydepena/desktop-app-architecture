# Importers

An importer turns one kind of source file into editor-ready data, checks it, and reports what it found.

## A registry by type

Importers are registered by file type (the **Registry** and **Strategy** patterns):

```
importers.register("png", "jpg", "psd", "exr"  -> TextureImporter)
importers.register("fbx", "gltf", "glb", "obj" -> ModelImporter)
importers.register("wav", "ogg", "flac"        -> AudioImporter)
importers.register("drawing"                   -> DrawingImporter)   // the application's own documents
```

Each runtime service provides the importers for its own data, and plugins can add more
([Plugins](../10-plugins.md)). An importer can produce several sub-assets from one file
([The asset database](02-asset-database.md)).

## Import settings are documents

How a file is imported is decided by its **import settings**: texture compression and filtering, pixels per unit, sRGB or
linear, which animations to keep, audio loop points, whether to stream. These settings:

- are stored in the asset's **sidecar file**
- are edited in an **inspector** ([Inspectors](../07-inspectors.md)), described by the data description like any other
  object
- are **documents**: edits have undo, unsaved changes are tracked, and saving writes the sidecar
  ([Documents](../02-documents.md))
- trigger a **reimport** when they change

Defaults come from **import presets**, applied by folder or file pattern ("everything under `ui/` imports as UI
textures without mipmaps").

## Validation

Importers check their input and report **problems**, attached to the asset in the problems list
([Diagnostics](../../runtime/05-diagnostics.md)):

- a texture larger than allowed, or not a size the target supports
- a model without texture coordinates, with too many bones, at the wrong scale
- clipping or silent audio, the wrong sample rate
- missing references inside the source (a model pointing at a texture that doesn't exist)

Problems don't always stop the import: warnings import with a marker; errors keep the previous good result in place.

## Deterministic output

The same source, settings and importer version must always produce **the same bytes**. No timestamps, random ids,
machine names or hash-map ordering in the output. Caching ([The derived data cache](04-derived-data-cache.md)) and
reproducible builds both depend on it.

## Previews and thumbnails

Importers (or companion generators) produce **thumbnails** and **previews** (a waveform, a turntable render of a model)
on worker threads, stored in the cache, for the asset browser ([Editor features](09-editor-features.md)).
