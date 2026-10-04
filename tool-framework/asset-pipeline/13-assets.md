# Assets

The data the pipeline itself uses, as opposed to the assets it processes.

## By stage

| Stage | Data it uses | What it holds |
|---|---|---|
| **Identity and import** | **sidecar files** | each asset's id, importer and import settings |
| | **import presets** | default settings by folder or file pattern ("UI texture", "music", "voice line") |
| **Cooking** | **platform profiles** | per target: texture formats, audio codecs, shader targets, maximum sizes, compression levels |
| | **root asset lists** | what the product starts from, so only reachable assets are cooked |
| **Packaging** | **package manifests and chunk rules** | which assets go into which archive or chunk (shared with [IO](../../runtime/io/09-assets.md)) |
| | **file ordering lists** | the load order recorded from play sessions |
| **Organization** | **labels and collections** | grouping and searching assets |
| | **folder rules** | presets, labels or restrictions applied by location |
| **Infrastructure** | **cache settings** | local cache size, shared cache locations |

## Generated, never committed

- the **asset database**
- the **derived data cache**, local and shared
- **cooked builds** and **packages**
- **thumbnails and previews**
- **build reports** (kept as build artifacts, not in the project)

All of them can be rebuilt from the committed data above ([Collaboration](10-collaboration.md)).

## Secrets

Credentials for shared caches, build servers and signing keys for packages are **secrets**: provisioned per machine or
per build environment, never committed with the project and never written into sidecars, presets or profiles.
