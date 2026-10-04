# Resources

Loading external data (images, sounds, fonts, shaders, other files a document refers to) **by reference**, so it
is loaded once, shared, and replaceable while in use.

Reading the files (logical paths, packages, asynchronous requests and their priorities) is handled one layer down, by
[IO](io/README.md); this page is about the loaded data and the handles that point to it.

## Handles

Code never holds the loaded data directly; it holds a **handle**:

```
interface Handle<T> {
    String path();          // or a stable id (below)
    State state();          // NOT_LOADED, LOADING, LOADED, MISSING
    T get();                // the data, or a placeholder while not loaded
}
```

Because everyone holds the handle, the cache can load, reload, unload or move the data underneath, and every user
sees the new data at once. This is the **proxy** pattern. One handle per resource is shared by all users (the
**flyweight** pattern).

- **Placeholders:** while loading or when the file is missing, `get()` returns a placeholder (a checkerboard
  image, silence) instead of null. Nothing has to check for null, and missing data is visible.
- **States** let the UI show progress and errors per resource.

## The cache

```
interface ResourceCache {
    <T> Handle<T> get(String path, Class<T> type);   // the handle, loaded or not
    void load(Handle<?> h);                           // now (blocking)
    Future<?> loadAsync(Handle<?> h);                 // in the background (see Jobs)
    void reload(Handle<?> h);                         // the file changed
    void unload(Handle<?> h);                         // free the data, keep the handle
    void move(String from, String to);                // the file was renamed: handles follow
    void close();                                     // free everything (scope closed)
}
```

Caches belong to a scope ([Services and scopes](02-services-and-scopes.md)): usually one for the app's built-in resources (read-only) and
one per workspace.

## Loaders and savers by type

A **registry** maps file extensions to loaders (and savers, for editable resources):

```
interface Loader<T, D> {
    List<String> extensions();
    D decode(byte[] bytes);          // any thread: parse the file, no GPU, no other resources
    T finish(D decoded, Cache c);    // main thread: upload to the GPU, resolve references to other resources
}
interface Saver<T> {
    List<String> extensions();
    byte[] encode(T resource);
}
```

Splitting loading in two lets the slow part (reading, decoding) run on worker threads while the part that needs
the main thread (GPU uploads, other resources) stays short. New types are added by registering a loader,
which is also how plugins add them.

## Stable ids instead of paths

Referring to resources by path breaks every reference when a file is moved or renamed. Robust editors give each
resource a **permanent id** (a GUID), stored in a small sidecar file next to it (`player.png.meta`) along with its
import settings:

```
{ "id": "7c1e...", "import": { "filter": "nearest", "pixelsPerUnit": 64 } }
```

Documents store the id; the cache maps ids to current paths. Moving a file (with its sidecar) changes nothing
for the documents that use it.

## Import settings

Raw files (an image, a sound) aren't changed by the editor, but how they're used is: filtering, compression,
looping, slicing. Those settings live in the sidecar, are editable documents themselves, and trigger a reload
when changed.

How source files become runtime data in the first place (importers, caching, cooking per platform) is the
[asset pipeline](../tool-framework/asset-pipeline/README.md), a tool-framework system whose results this cache loads.

## Hot reload

Watch the resource files; when one changes on disk, reload it through its handle. Users see the change without
restarting. Ignore changes the app made itself, and debounce (editors often write a file several times in a row).

## Rules

- The cache never writes files on its own; saving goes through documents ([Documents](../tool-framework/02-documents.md)).
- Dependencies (a material's textures, a font's glyph images) are loaded through the same cache, and cycles are detected.
- Unloading something still in use only frees the data: handles stay valid and show the placeholder.
