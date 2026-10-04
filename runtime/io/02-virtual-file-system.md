# The virtual file system

The key idea of an IO service: **the application uses logical paths, and the virtual file system (VFS) maps them to
wherever the data actually is.**

```
content:/textures/grass.png   ->  <install>/content/textures/grass.png            (development: loose files)
                              ->  <install>/content.pak, entry "textures/grass.png" (shipped: a packed archive)
user:/settings.json           ->  %APPDATA%/App/settings.json
project:/world/level1.world   ->  <the open project folder>/world/level1.world
cache:/thumbnails/abc.png     ->  %LOCALAPPDATA%/App/Cache/thumbnails/abc.png
mem:/test/fixture.json        ->  an in-memory file (tests)
```

## Mount points and backends

- A **mount point** is a prefix (`content:`, `user:`, `project:`, `cache:`).
- Each mount is backed by a **backend**: a folder on disk, a packed archive, memory, a network or cloud store.
  Backends share one interface and are interchangeable (the **Strategy** pattern).
- Mounts are created at startup (install content, user folders) or when something opens (a project's folder when a
  workspace opens, unmounted when it closes).

```
interface Backend {
    boolean exists(String path);
    FileInfo info(String path);                     // size, modified time, hash if known
    InputStream openRead(String path);
    OutputStream openWrite(String path);            // throws if the backend is read-only
    List<String> list(String directory);
    boolean isWritable();
}
```

## Read-only and writable mounts

Shipped content is **read-only**. Writes go to `user:`, `cache:` or the project. A write to a read-only mount fails
immediately with a clear error, instead of half-working on a developer's machine and failing for users.

## Overlays and search order

Several backends can be mounted on one prefix, and searched in order:

```
content:  ->  1. mods/MyMod/      (searched first: overrides)
              2. patch-02.pak
              3. patch-01.pak
              4. content.pak      (the base content)
```

The first backend that has the file wins. This is how **patches, downloadable content and mods** replace files
without touching the originals. It is a **chain of responsibility**, or a layered **composite** of backends.

Listing a directory on an overlaid mount merges the listings of every layer, with higher layers hiding lower ones.

## Why a VFS

- **Portable data:** code, saved files and settings never contain machine-specific absolute paths. A project moved
  to another folder or machine keeps working.
- **One path, two forms:** development uses loose files (easy to edit and hot-reload); shipping uses packed archives
  (fast to load). The paths in code and data are the same.
- **Testable:** tests mount an in-memory backend in place of real folders.
- **One place for policy:** read-only rules, case normalization and access checks live in one layer.

The VFS is a **Facade** over all storage, and a logical path is a **stand-in for a physical location**, the same
indirection that handles provide for GPU resources and actions provide for devices.

## Rules

- Logical paths in data, physical paths only inside the IO service.
- Logical paths are **normalized**: forward slashes, no `..`, and one agreed case convention (lowercase is common),
  so a file found on Windows is also found on Linux.
- Paths from outside (user input, downloaded data, mods) are **untrusted**: a path like `content:/../../system` must
  never escape its mount.
