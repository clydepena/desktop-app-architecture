# Using the asset pipeline

How the rest of the editor and the build tools use the pipeline.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Database** | find and inspect assets | `find(id)`, `findByPath(path)`, `query(type, label)`, `subAssets(id)` |
| **Import** | bring files in, change settings | `import(path, settings?)`, `reimport(id)`, `settings(id)` (an editable document) |
| **References** | dependencies, both ways | `dependencies(id)`, `referencers(id)` |
| **Safe operations** | rename, move, duplicate, delete | `move(id, newPath)`, `duplicate(id)`, `delete(id) -> Result<referencedBy>` |
| **Thumbnails and previews** | show assets | `thumbnail(id, size) -> Future<Image>` |
| **Cooking** | produce runtime data | `cook(target, roots) -> BuildReport` |
| **Packaging** | produce shippable archives | `package(target, manifest) -> Packages` |
| **Cache** | inspect and manage | hit rates, size, `clearLocal()`, shared cache settings |
| **Events** | react to changes | `assetImported(id)`, `assetMoved(id)`, `importFailed(id, problems)` |

## How each part of an application uses it

| Part | How it uses the pipeline |
|---|---|
| **Resources** | load cooked data by id; in the editor, load freshly imported data and swap it in when a reimport finishes ([Resources](../../runtime/07-resources.md)) |
| **IO** | provides file watching, the `cache:` mount, streams and hashing; receives package manifests ([IO](../../runtime/io/README.md)) |
| **Jobs** | run imports and cooks in parallel, with priorities, progress and cancellation ([Jobs](../../runtime/06-jobs.md)) |
| **Documents and undo** | import settings are small documents with undo; asset operations (move, delete, duplicate) are undoable ([Documents](../02-documents.md)) |
| **Rendering** | provides texture, model and shader importers and cookers ([Rendering assets](../../runtime/rendering/08-assets.md)) |
| **Audio** | provides clip and bank importers and cookers ([Audio assets](../../runtime/audio/09-assets.md)) |
| **Localization** | string extraction and string table building are pipeline steps ([The translation pipeline](../../runtime/localization/08-translation-pipeline.md)) |
| **Scripting** | compiling scripts to bytecode is a cook step ([Scripting assets](../../runtime/scripting/12-assets.md)) |
| **Input, UI, time** | their definition assets (bindings, screens, schedules) are imported and validated like any other |
| **Diagnostics** | import and cook problems in the problems list, linked to the asset; build reports in the build output |
| **UI toolkit** | the asset browser, import settings dialogs, the dependency viewer ([Editor features](09-editor-features.md)) |
| **Networking** | the shared derived data cache is a request/response service ([Request/response services](../../runtime/networking/06-request-response-services.md)) |
| **Build machines** | run the same pipeline from the command line: import, cook, package, validate |
