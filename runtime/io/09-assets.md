# Assets

IO's own assets are mostly **layout and packaging rules**. IO is also the layer that **stores** every other
service's assets, but those are described in their own folders: [Rendering](../rendering/08-assets.md),
[Audio](../audio/09-assets.md), [Networking](../networking/11-assets.md), [Input](../input/11-assets.md),
[Time](../time/11-assets.md), [Windowing](../windowing/09-assets.md), [Memory](../memory/11-assets.md),
[Scripting](../scripting/12-assets.md), [Localization](../localization/12-assets.md) and [UI](../ui/12-assets.md).

The [asset pipeline](../../tool-framework/asset-pipeline/README.md) produces the cooked files and package manifests
that IO stores and reads.

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Virtual file system** | **mount tables** | which prefixes map to which backends, per platform and build type (development: loose folders; shipping: archives) |
| | **overlay rules** | search order for patches, downloadable content and mods |
| **Packages** | **package manifests** | which files go into which archive or chunk |
| | **compression and encryption settings** | per group of files: fast or strong compression, encrypted or not |
| | **file ordering lists** | the order files are loaded in, recorded from play sessions, used to lay archives out for sequential reads |
| **Scheduler and streaming** | **streaming settings** | bandwidth budgets and priorities per category, unload delays |
| | **chunk layouts** | how a large world is divided into streamable regions |
| **Formats** | **binary format schemas** | field layouts for binary files, often shared with the data description |
| **Tools** | **ignore lists** | files never packaged (sources, notes, temporary files) |
| | **import rules** | which importer and settings apply to which files |
| | **content hash caches** | hashes of source files, so unchanged files are not imported again |

## What is not an asset

- **The user's own files** (their documents, exports): IO stores them but doesn't own them.
- **Caches:** always rebuildable, never shipped, safe to delete.
- **Temp files:** deleted after use.
- **Secrets:** archive encryption keys and signing keys belong to the build pipeline, provisioned per build, never
  committed with the project's files.

## Generated vs authored

Some IO data is **generated** rather than written by people: file ordering lists come from recorded sessions, package
manifests are often generated from dependency information, content hash caches from the files themselves. Generated
data is rebuilt by tools and kept out of hand editing; the rules that drive the generation are the authored assets.
