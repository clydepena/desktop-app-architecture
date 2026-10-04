# Collaboration

How the pipeline works when many people change assets at the same time.

## What goes into version control

| Commit | Never commit |
|---|---|
| source assets | the asset database |
| sidecar files (ids, import settings) | the derived data cache |
| project and pipeline settings, presets, platform profiles | cooked builds and packages |
| editor documents (levels, materials, prefabs) | thumbnails and previews |

The right-hand column is generated, and rebuilt or downloaded on demand
([Source and runtime data](01-source-and-runtime-data.md)).

## Binary files can't be merged

Two people changing the same PSD or FBX at once can't have their work combined: one change is lost. So:

- **lock binary files** before editing them; most version control systems used for games support locking
- keep **sidecars and editor documents text-based**, so they can be diffed, reviewed and merged
- split large editor documents (one huge level) into smaller ones (level sections, prefabs), so people rarely need the
  same file at the same time

## Missing or conflicting sidecars

- A source committed **without** its sidecar gets a new id on every machine that imports it: references to it then
  differ per machine. Pipelines warn about sources without sidecars, and version control hooks can refuse such commits.
- Two people adding the **same file** at once create two ids for one asset. The pipeline reports duplicate paths, and
  one id is kept, with references to the other redirected.

## Build machines and the shared cache

- **Build machines** import and cook every commit for every platform, and fill the **shared cache**
  ([The derived data cache](04-derived-data-cache.md)).
- Teammates then **download** results instead of computing them: after pulling, most assets come straight from the
  cache.
- The same machines run validation: new problems (oversized textures, missing references, assets over budget) fail the
  build, so they are fixed before they reach everyone.

## Credentials

Shared cache servers and build machines need credentials. Those are **secrets**: provisioned per machine, never
committed with the project ([Assets](13-assets.md)).
