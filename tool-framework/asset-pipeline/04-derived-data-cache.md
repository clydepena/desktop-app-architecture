# The derived data cache

The heart of a fast pipeline: **never compute the same result twice.**

## Content-addressed keys

Every imported or cooked result is stored under a **key** computed from everything that affects it:

```
cache key = hash( source bytes,
                  import settings,
                  importer (or cooker) version,
                  target platform,
                  cache keys of the dependencies )
```

Before running an importer, the pipeline computes the key and looks it up. If the result exists, it is used; only
missing keys are computed. This is **memoization**, applied to builds.

## Why not timestamps

| Timestamps | Content hashes |
|---|---|
| touching a file without changing it triggers a rebuild | an unchanged file has the same key: nothing to do |
| switching branches back and forth rebuilds everything twice | both versions' results stay cached and are reused |
| a wrong clock or a copied file can skip a needed rebuild | any change to anything that matters changes the key |
| can't be shared between machines | the same key means the same result on every machine |

Hashing every source file is not free, so the database remembers each file's size, modification time and hash, and
re-hashes only files whose size or time changed.

## The importer version is part of the key

Fixing a bug in the texture importer, or changing its output, means bumping its **version**. That changes the keys of
exactly the results it produced, so they are rebuilt, and nothing else is.

## Local and shared caches

- The **local cache** lives under the IO service's `cache:` mount ([The virtual file system](../../runtime/io/02-virtual-file-system.md)).
- A **shared cache** (a server or network folder) is filled by build machines and teammates. A new team member, or a
  fresh checkout, **downloads** results by key instead of computing them for hours.
- Lookups go local first, then shared; new results are written to both (or only by trusted build machines, so a broken
  local importer can't poison everyone's cache).

## Always deletable

Losing the cache costs time, never data. Deleting it is a safe fix for any suspected cache problem, and the cache has a
size limit with least-recently-used eviction ([Writing safely](../../runtime/io/06-writing-safely.md)).
