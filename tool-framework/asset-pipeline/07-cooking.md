# Cooking

Turning editor-ready data into exactly what one target platform loads. Import happens once per source; cooking happens
once per source **per platform**.

## Per-platform processing

| Asset | Cooked differently per platform |
|---|---|
| textures | the compression formats each GPU family supports (BC on desktop, ASTC on phones), maximum sizes, mip levels |
| shaders | compiled for each graphics API and its variants |
| audio | codecs and quality per platform, streaming vs in memory |
| models | vertex formats, index sizes, levels of detail |
| scripts | bytecode, or compiled ahead of time where runtime compilation isn't allowed |

Each runtime service provides the **cookers** for its data, registered like importers. **Platform profiles**
([Assets](13-assets.md)) hold the per-target choices.

## Stripping

Data that only the editor needs is removed: source file references, editor-only metadata and helper objects, debug
names where wanted, unused import variants. The shipped product never contains the editor's view of an asset.

## Only what is reachable

Cooking everything in the project ships test assets, abandoned experiments and editor-only content. Instead, the
cooker starts from **root assets** (the startup world, the levels, always-loaded UI, assets loaded by name at runtime)
and follows references through the asset graph. Only what can be reached is cooked and shipped.

Assets loaded by name or path at runtime (rather than by reference) are invisible to this walk, which is one more
reason to refer to assets by id: anything loaded indirectly must be listed as a root explicitly.

## The build report

Each cook produces a report:

- the size of every asset, per category and per package
- **why** each asset was included: the chain of references from a root to it
- what changed since the last build, and its effect on size
- problems found during cooking

When a build is too big, the report answers "what is taking the space, and who pulled it in?".

## Cooking without the editor

The same pipeline runs from the command line on build machines: no UI, the same importers and cookers, the same cache.
This is only possible because the pipeline depends on the runtime services and never on the editor's UI
([Coupling](12-coupling.md)).
