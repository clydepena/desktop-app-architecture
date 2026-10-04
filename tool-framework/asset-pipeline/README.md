# The asset pipeline

How the files people make (images, models, recordings, drawings, scripts) become the data the product loads.

Every runtime service's assets page mentions it: "source formats vs runtime formats", "import settings in a sidecar",
"an import step converts...". This series describes the system that does it. It belongs to the **tool framework**:
only editors and build tools run it. Its mirror image at runtime is [Resources](../../runtime/07-resources.md), which
loads its results.

Its special problems are **scale and change**: thousands of files, many people editing them, several target
platforms, and conversions that can take minutes. The goal is that changing one texture never means waiting for
everything to rebuild.

## The stages

```
+------------------------------------------------------------------+
| Source assets      what people edit: PNG, PSD, FBX, WAV, .drawing, |  in version control
|                    scripts; plus import settings (sidecar files)   |
+------------------------------------------------------------------+
| Asset database     identity (stable ids), paths, types,            |  the index of everything
|                    dependencies, labels                            |
+------------------------------------------------------------------+
| Import             importers per type: validate, convert to an     |  editor-ready data
|                    editor representation, make thumbnails          |
+------------------------------------------------------------------+
| Cooking            per target platform: compress, compile shaders, |  runtime-ready data
|                    pack atlases, strip editor-only data            |
+------------------------------------------------------------------+
| Derived data cache results keyed by content hash; local and shared |  never rebuild twice
+------------------------------------------------------------------+
| Packaging          manifests, chunks, archives, file ordering      |  hands off to IO
+------------------------------------------------------------------+
| Runtime            resources load cooked data by id                 |  the shipped product
+------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Source and runtime data](01-source-and-runtime-data.md) | why both exist, and what goes in version control |
| 2 | [The asset database](02-asset-database.md) | stable ids, sidecar files, references by id, sub-assets |
| 3 | [Importers](03-importers.md) | the importer registry, import settings as documents, validation, determinism |
| 4 | [The derived data cache](04-derived-data-cache.md) | content-addressed keys, importer versions, local and shared caches |
| 5 | [Dependencies and incremental builds](05-dependencies-and-incremental-builds.md) | the asset graph, rebuilding only what changed |
| 6 | [Background import](06-background-import.md) | file watching, prioritized jobs, bulk changes, hot reload |
| 7 | [Cooking](07-cooking.md) | per-platform processing, stripping, reachability, build reports |
| 8 | [Packaging and versions](08-packaging-and-versions.md) | handing off to IO packages; importer, asset and sidecar versions |
| 9 | [Editor features](09-editor-features.md) | the asset browser, reference-safe operations, missing references |
| 10 | [Collaboration](10-collaboration.md) | what to commit, locking binary files, build machines and shared caches |
| 11 | [Using the asset pipeline](11-using-the-asset-pipeline.md) | the public interface, and how each part of an app uses it |
| 12 | [Coupling: what may depend on the pipeline](12-coupling.md) | tool-only by design; determinism and clean-vs-incremental tests |
| 13 | [Assets](13-assets.md) | sidecars, presets, platform profiles, root lists, what is generated |

## Patterns in the asset pipeline

| Pattern | Where |
|---|---|
| Pipes and filters | import, then cook, then package |
| Registry + Strategy | importers and cookers chosen per asset type |
| Repository | the asset database over the file system |
| Proxy | references by id, resolved through handles |
| Memoization / content-addressed caching | the derived data cache |
| Dependency graph + topological sort | incremental builds, as in build systems like Make or Bazel |
| Observer | file watching triggering reimports |
| Command | import and cook jobs |
| Composite / Factory | one source file producing several sub-assets |

## Related

- [Resources](../../runtime/07-resources.md): the runtime side, loading what the pipeline produces.
- [IO](../../runtime/io/README.md): file watching, the cache folder, hashing, and the package formats.
- [Jobs](../../runtime/06-jobs.md): imports and cooks in parallel.
- [Persistence](../03-persistence.md): format versions and migrations, shared with assets.
- The assets page of each runtime service, for what its importers and cookers produce:
  [rendering](../../runtime/rendering/08-assets.md), [audio](../../runtime/audio/09-assets.md),
  [localization](../../runtime/localization/12-assets.md), [scripting](../../runtime/scripting/12-assets.md),
  [input](../../runtime/input/11-assets.md), [UI](../../runtime/ui/12-assets.md).
