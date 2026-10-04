# Dependencies and incremental builds

Assets depend on other assets. Rebuilding only what a change actually affects requires knowing those dependencies
exactly.

## The asset graph

```
level "Forest"
  -> prefab "Tree"
       -> model "tree.fbx"
       -> material "Bark"
            -> texture "bark.png"
            -> shader "lit.shader"
                 -> include "lighting.inc"
```

Two kinds of edges:

- **references:** an asset refers to another by id (a material to its texture)
- **build inputs:** a file read while importing (a shader's include files, a model's external textures)

## Importers report what they read

Instead of guessing dependencies from file contents, each importer **reports** every asset and file it used. The
database records them, so the graph is exact: a change to `lighting.inc` rebuilds every shader that included it, and
nothing else.

## Incremental builds

When something changes:

1. mark it and everything that depends on it (directly or indirectly) as **out of date**
2. rebuild them in **topological order**: dependencies before what depends on them
3. for each, compute the cache key first; if the result already exists ([The derived data cache](04-derived-data-cache.md)),
   reuse it instead of rebuilding

This is how general build systems such as Make and Bazel work, applied to content. Independent assets are rebuilt in
parallel on the jobs system ([Jobs](../../runtime/06-jobs.md)).

## Cycles

A cycle (A depends on B, B depends on A) can't be built in any order. It is reported as an error with the chain of
assets that forms it, so someone can break it.

## What changes propagate

Not every change to a dependency affects its dependents. A texture's **pixels** changing doesn't change a material that
only references the texture by id: the material's cooked data stays the same, only the texture is rebuilt. Keeping
"refers to" separate from "built from" avoids rebuilding half the project after every texture edit.
