# Packaging and versions

## Handing off to packages

The final step turns cooked results into what ships, using the formats described in
[Packages](../../runtime/io/03-packages.md):

- **package manifests:** which cooked assets go into which archive or chunk (often generated from the reachability walk:
  each level and what only it uses in its own chunk, shared assets in a common one)
- **compression** per group of files
- **file ordering:** the order recorded from play sessions, so loading reads sequentially
- **patches:** only the archives or entries that changed since the last release, mounted above the old ones

The pipeline produces the inputs; the IO service defines and reads the formats.

## Importer and cooker versions

Every importer and cooker has a **version**, part of every cache key it contributes to
([The derived data cache](04-derived-data-cache.md)). Changing what one produces means increasing its version, which
rebuilds exactly its results and nothing else.

## Asset format versions

Assets created in the editor (materials, prefabs, levels, the application's own documents) have **format versions**
like any document ([Persistence](../03-persistence.md)):

- older assets load, and are **migrated** in memory to the current format
- they are written in the new format only when someone edits and saves them, never just because they were opened
- a project can be upgraded in one deliberate step (a "resave all" tool) when the team decides to

## Sidecar versions

Sidecar files are versioned too. They are small and text-based, so their diffs stay readable and their merges usually
succeed ([Collaboration](10-collaboration.md)).
