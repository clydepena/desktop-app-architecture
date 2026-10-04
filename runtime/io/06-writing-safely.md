# Writing safely

Reading a broken file is an error message; writing a broken file is lost work. Writing gets its own rules.

## Atomic writes

Never overwrite a file in place. Write the new content beside it, then swap:

```
1. write to   file.tmp     (in the same folder, so step 3 is a rename, not a copy)
2. flush it to disk
3. rename file.tmp over file
```

If anything fails before step 3, the original is untouched. The IO service offers this as one call,
`writeAtomic(path, bytes)`, so every caller (documents, settings, caches, UI state) gets it right without repeating
it. [Persistence](../../tool-framework/03-persistence.md) builds its document saving on top of it.

## Flushing

Operating systems keep written data in memory for a while before it reaches the disk. A power cut in that window
loses data that the app believed was saved. **Flushing** (`fsync` and its equivalents) forces it out. It is slow, so
it is used where it matters: document saves, settings, the files an atomic write renames.

## Expect failure

Every write can fail, and must report **why**:

| Failure | Typical cause |
|---|---|
| **disk full** | large saves, caches growing without limit |
| **permission denied** | writing to the install folder, a read-only file, a protected folder |
| **file locked** | open in another program: an editor, antivirus, a cloud sync client |
| **path too long / invalid** | long project paths, characters not allowed on that file system |
| **device gone** | a USB drive or network share disconnected mid-write |

An app that ignores a failed write is losing the user's data silently. Report it, keep the data in memory, and offer
another location or a retry.

## Cloud-synced folders and network drives

Projects often live in folders synced by OneDrive, Dropbox or similar, or on network shares:

- files can be **locked briefly** while the sync client uploads them: retry with a short backoff
- **renames may not be atomic**, and the sync client may create conflict copies
- changes can **appear from other machines** at any time: file watching reports them, and open documents ask
  whether to reload ([Documents](../../tool-framework/02-documents.md))

## Locking

A project opened twice (by two instances of the app, or by two people on a shared drive) can overwrite itself. A
**lock file** created when the project opens (holding who opened it and when) lets the second opener warn ("already
open on machine X") or open read-only. Lock files must be cleaned up on close, and stale ones (after a crash)
recognized by age or by checking whether the owner still runs.

## Cleaning up

- Temp files are deleted after use, and leftovers from crashes are removed at the next start.
- Caches have size limits and evict the oldest entries; deleting the whole cache must always be safe.
- Logs rotate: a few files of bounded size, not one file growing forever.
