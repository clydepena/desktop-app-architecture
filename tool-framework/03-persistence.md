# Persistence

Turning documents into files and back, without ever losing the user's work.

The files themselves (paths, the virtual file system, atomic writes, streams, compression) are handled one layer
down, by [IO](../runtime/io/README.md); this page is about what goes into a document file and how it evolves.

## Keep the file format separate from live objects

Live objects are shaped for running (caches, references, GPU handles); files are shaped for storing (plain
values, ids instead of references, a version number). Keep a **file model** class per format (a DTO, data
transfer object) and convert between the two:

```
live document  --toFile()-->  DrawingFile { version, shapes... }  --serialize-->  bytes
bytes  --parse-->  DrawingFile  --toLive()-->  live document
```

This lets the format change without touching the live classes, and the reverse.

## Versions and migrations

Every file starts with a format version:

```
{ "formatVersion": 3, ... }
```

On load:

- **older version:** run migrations, one per version step (1->2, 2->3), then load as current.
- **newer version:** refuse with a clear message ("made with a newer version of the app").
- **no version:** the oldest format; treat as version 0 or 1.

Migrations are small, tested functions on the file model, never on live objects. Files are rewritten in the
current version only when the user saves, never on load.

## Atomic save

Never write over the real file directly. A crash, a full disk or a power cut halfway through would leave a broken
file, and the old version would be gone too.

```
1. write the new content to   file.tmp   (same folder, so step 2 is a rename, not a copy)
2. flush it to disk
3. rename file.tmp over file           (atomic on the same file system)
```

If step 1 or 2 fails, the original file is untouched. Where atomic rename isn't available, fall back to
a plain replace and keep a `.bak` of the previous file. The IO service provides this as one call
([Writing safely](../runtime/io/06-writing-safely.md)).

## Autosave and crash recovery

Two different features, often confused:

| | Autosave to the real file | Recovery copies |
|---|---|---|
| What | saves the document every N minutes | writes a backup copy somewhere else every N minutes |
| User's file | changed without asking | untouched until the user saves |
| Common in | note apps, cloud apps | desktop editors (Office, Photoshop, IDEs, game editors) |

Editors usually choose recovery copies:

- every N minutes, and only if dirty: write each dirty document to a recovery folder (in the user's data
  folder, not the workspace), with a small index file (which document, which original path, when).
- on a normal close: delete the recovery copies.
- on the next start, if recovery copies exist (the app crashed): offer "Restore unsaved work?" per document.

Write recovery copies on a background thread when serialization is slow ([Jobs](../runtime/06-jobs.md)), from a snapshot taken on the
main thread.

## Snapshots in memory

The same serialization serves short-lived snapshots: copy-paste, duplicate, a "preview" or "run" mode (run the
content, then put it back exactly as it was). Serialize to memory, restore from memory. This is the **memento** pattern, and a
free test of the save format: anything that doesn't survive a snapshot won't survive a save either.

## Identity across saves

References between objects in a file need ids that **stay the same across save and load**. Objects given new
ids on every load break everything that refers to them: undo commands, links between objects, linked copies,
network sync. Store a stable id in the file (a GUID or a per-document counter that is saved with it).
