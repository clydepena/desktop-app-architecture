# Documents

The center of every editor. A **document** is the unit the user opens, edits, saves and closes:

```
document = data + its file + its unsaved state + its undo history
```

A drawing, an image, a text file, a spreadsheet, a settings file: each is a document. Almost every other system
refers to documents: undo histories belong to one, actions apply to the active one, saving and closing go
through them.

## The document object

```
interface Document {
    DocumentType type();
    Path file();                      // null until first saved ("Untitled")
    String displayName();             // "poster.drawing", "Untitled 2"
    boolean isDirty();                // changed since last save or load
    UndoHistory history();            // see Undo
    Scope scope();                    // services living as long as the document
    void save() throws IOException;   // to file()
    void saveAs(Path path) throws IOException;
    void revert() throws IOException; // throw away changes, reload from file()
}
```

The domain data (the actual drawing, image or text) sits behind this interface. The framework only ever uses the
interface.

## The document manager

Tracks every open document and owns the operations on them:

| Operation | Rules |
|---|---|
| **New** | creates an untitled document of a type; first Save asks for a path |
| **Open** | already open? focus it instead of opening twice. Otherwise find the type by extension, load, add |
| **Save** | write to its file; untitled -> Save As |
| **Save As** | ask for a path, write there, the document now *is* that file |
| **Save All** | every dirty document; failures are reported and stay dirty (see below) |
| **Close** | dirty? ask Save / Don't save / Cancel; then dispose its scope |
| **Revert** | confirm, then reload from the file and clear the history |
| **Reload** | the file changed on disk (below) |

### Dirty tracking

"Dirty" means *different from the file*. The most robust way to know: let the **undo history** decide ([Undo](04-undo.md)).
The history remembers its position at the last save; dirty means "not at that position". Undoing back to the
saved state then makes the document clean again, as users expect.

Changes that bypass the history (rare: an external tool rewrote data in memory) mark the document dirty
directly and make the saved position unreachable.

### Save All and failures

```
List<Failure> saveAll() {
    for each dirty document:
        try { document.save(); }               // success: no longer dirty
        catch (e) { failures.add(document, e) } // stays dirty, can be retried
    return failures;                            // empty: everything saved
}
```

The **unit of work** pattern: collect what changed, write it together, report what failed. Never return "null
for success"; never drop the reason a save failed.

### Closing with unsaved changes

One dialog for every way of closing (a document's tab, Close Workspace, Quit, the OS shutting down):

```
"Save changes to these documents?"
  - poster.drawing
  - notes.txt
[Save All]  [Don't Save]  [Cancel]
```

- **Save All** closes only if every save succeeded; otherwise the dialog stays, showing the failures.
- **Cancel** cancels the whole close (the app keeps running).

### Files changed on disk

Watch the open documents' files. When one changes outside the app:

- not dirty: reload silently (or ask, if reloading is expensive)
- dirty: ask "The file changed on disk. Reload (lose your changes) / Keep mine (overwrite on next save)?"

Ignore changes the app itself just made (remember the time or content hash of its own last save).

## Document types

A **registry** maps file extensions to types; each type knows how to create, load and save its documents and
which views can show them:

```
interface DocumentType {
    String name();                     // "Drawing"
    List<String> extensions();         // ["drawing"]
    Document create();                 // New
    Document load(Path file);          // Open
    List<ViewFactory> views();         // which panels can show it
}
```

New document types are added by registering, which is also how plugins add them ([Plugins](10-plugins.md)).

## Documents vs resources

Not everything with a file is a document:

| | Document | Resource |
|---|---|---|
| Opened by | the user, to edit it | the app, because something references it |
| Has | an undo history, unsaved state, a tab/window | a handle, a cache entry |
| Example | the drawing being edited | an image placed in the drawing |

The same file can be both: an image is a resource while a drawing places it, and a document while the image
editor has it open. Opening it as a document should share the loaded data with its resource handle, so
edits show up everywhere.

## Several views of one document

One document, many views: a drawing shown in a canvas *and* a layers list; one text file in two split panes.
Views never own the data. They read the document and send edits as commands ([Undo](04-undo.md)). This is the
**model-view** separation; with it, adding a view never touches the document.
