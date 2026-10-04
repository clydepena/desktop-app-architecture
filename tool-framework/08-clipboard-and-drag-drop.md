# Clipboard and drag-and-drop

Moving data between places: within a document, between documents, between the app and other apps. Both features
use the same model, so they're one system.

## Typed data, several formats at once

Copying puts the same data on the clipboard in **several formats**, from richest to simplest; the receiver picks
the best one it understands:

```
copy 3 selected objects ->
    "application/x-myapp-objects"   the full objects (serialized, as in a save)
    "text/plain"                    their names, one per line
```

```
interface DataPackage {
    Set<String> formats();
    byte[] get(String format);
}
```

- Pasting into the same app uses the rich format: the objects come back complete.
- Pasting into a text editor gets the names.
- Copying an image region offers the app's format plus a standard image format.

The rich format is usually the same serialization as saving ([Persistence](03-persistence.md)), so copy/paste works for everything
that can be saved.

## Paste

- Paste creates **new** objects: new ids, references between pasted objects re-linked to the new copies,
  references to things outside the paste kept or cleared (a rule per reference type).
- Paste is one undo step (a composite command, [Undo](04-undo.md)), and selects what was pasted.
- **Where it goes:** at the same position (paste in place), offset so it's visible, or under the mouse. Pick one
  default and offer the others as separate actions.
- Cut = copy + delete, as one undo step.

## Drag-and-drop

The same packages, moved by the mouse:

- The **source** offers a package when a drag starts (and a preview image).
- Each **target** under the mouse says whether it accepts one of the formats, and what would happen (move, copy,
  link); the cursor shows it.
- Dropping runs the target's action, as an undo command.
- **External drops:** files dragged from the OS arrive as a list of paths: import them, or open them as
  documents ([Documents](02-documents.md)).
- **Dragging out:** offer a file format so other apps can receive it (export on drop).

## Rules

- Reading the clipboard can be slow or fail (another app holds it): never block the UI on it.
- Don't put huge data on the clipboard eagerly; offer it lazily (formats produced only when asked) where the
  platform allows.
