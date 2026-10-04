# Using the IO service

How the rest of an application uses storage: through logical paths and asynchronous requests, never through raw
platform calls. Completions and file-watch events come back to the caller's worker or the main thread; the IO
service never calls into application code otherwise.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Paths** | logical paths, normalized | `Path.of("content:/textures/grass.png")`, `resolve`, `relativize`, `extension()` |
| **Mounts** | where logical paths lead | `mount("project:", FolderBackend(root))`, `mountArchive("content:", pak, priority)`, `unmount(prefix)` |
| **Reading** | whole files or ranges, asynchronously | `read(path, priority) -> Future<Bytes>`, `readRange(path, offset, length)` |
| **Writing** | safely | `writeAtomic(path, bytes) -> Future`, `append(path, bytes)`, `delete(path)`, `move(from, to)` |
| **Streams** | sequential access | `openRead(path) -> InputStream`, `openWrite(path)`, wrapped with buffering, compression, hashing |
| **Listing** | directories and archives | `list(dir)`, `walk(dir, filter)`, `exists(path)`, `info(path) -> size, modified time, hash` |
| **Watching** | changes on disk | `watch(dir, handler)`: created, modified, deleted, renamed (debounced) |
| **Locations** | well-known folders | `userData()`, `cache()`, `temp()`, `documents()` |
| **Packages** | open and build archives | `openArchive(file)`, `PackageBuilder` (used by tools) |
| **Requests** | control in-flight work | `cancel(request)`, `setPriority(request, p)`, statistics |

## How each part of an application uses it

### The lifecycle

Creates the VFS early (settings and logging need it), mounts the install content and the user folders, mounts a
project when a workspace opens and unmounts it when it closes, and waits for pending writes before exiting
([Lifecycle](../01-lifecycle.md)).

### Resources

Read asset files through logical paths and the scheduler; decoding runs on workers. **Hot reload** comes from file
watching on writable mounts ([Resources](../07-resources.md)).

### Rendering, audio and input

Never open files. They receive decoded data from resources. **Audio streaming** issues urgent range reads from the
audio engine's worker threads; the audio thread itself only reads the buffer those workers fill.

### Persistence and documents

Use atomic writes, streams and format helpers, through logical paths such as `project:/...`
([Persistence](../../tool-framework/03-persistence.md)).

### Settings, recent lists and UI state

Live under `user:` ([Settings](../04-settings.md)). **Caches** (thumbnails, imported assets, compiled shaders) live
under `cache:` and may be deleted at any time.

### Diagnostics

Log files under `user:` or `cache:`, with rotation. Crash reports are written with the simplest, most robust path
available, since the application may be in a bad state when they are written
([Diagnostics](../05-diagnostics.md)).

### Networking

Downloads stream straight into `cache:` or a staging folder, are **verified by hash**, then mounted or moved into
place ([Request/response services](../networking/06-request-response-services.md)).

### Plugins

Discovered by listing plugin folders, and mounted as overlays so they can add or override content
([Plugins](../../tool-framework/10-plugins.md)).

### Editors

- **A file / asset browser:** listing, watching, thumbnails generated on workers into `cache:`.
- **An import pipeline:** source files turned into runtime formats, cached by **content hash**, so unchanged files
  are never imported twice.
- **Packaging tools** that build archives and file ordering for shipping.
- **Drag and drop** of external files into a project: copied in, then imported.
