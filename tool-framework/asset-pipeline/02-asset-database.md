# The asset database

The index of every asset in a project: what it is, where it is, how to import it, and what it depends on.

## Stable ids in sidecar files

Every asset gets a **stable id** (a GUID) when it first enters the project. The id is stored in a small **sidecar
file** next to the source, together with how to import it:

```
hero.png
hero.png.meta        { "id": "7c1e9a...", "importer": "texture", "settings": { "compression": "high", "mipmaps": true } }
```

This is the stable-id idea from [Resources](../../runtime/07-resources.md), put into practice. The sidecar travels with
the file: moving or renaming both keeps the id.

## References use ids, not paths

A material refers to its texture by id; a level refers to its prefabs by id. Renaming or moving a file therefore breaks
nothing: the database maps the id to the file's new path. Paths are only for people (browsing, searching); content
refers to ids.

## What the database holds

For every id:

- the current **path** and **type**
- the **importer** and its **settings**
- **dependencies**: the ids it references, and the files its import read
- **labels** and collections, for organizing and searching
- the **cache keys** of its imported and cooked results ([The derived data cache](04-derived-data-cache.md))

## The database is a cache too

The database is an **index** built from the sidecars and sources, the **Repository** pattern over the file system. It
is stored locally for speed, never committed, and rebuilt from the sidecars when it is missing or out of date. The
sidecars are the truth.

## Sub-assets

One source file can contain several assets: a model file holds meshes, materials, animations and a skeleton; a sprite
sheet holds many sprites. Each becomes a **sub-asset** with its own id, derived from the source's id and a stable
name inside it:

```
character.fbx            id 51b0...
  mesh "Body"            id 51b0.../mesh/Body
  material "Skin"        id 51b0.../material/Skin
  animation "Run"        id 51b0.../animation/Run
```

Sub-asset ids must stay stable when the source is re-exported, so they are based on names, not positions in the file.
Renaming something inside the source tool is the one change that breaks references to it, and the pipeline reports
those references as missing ([Editor features](09-editor-features.md)).

## Files without sidecars

A file added outside the editor (copied in, pulled from version control without its sidecar) gets a sidecar on
discovery, with a new id and default settings from **import presets** ([Assets](13-assets.md)). A sidecar whose source
is gone is reported, not silently deleted, since the source may come back with the next update.
