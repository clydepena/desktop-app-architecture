# Undo

Every change to a document is a **command**: an object that knows how to apply and reverse one edit. The
history stores commands, so the user can step back and forward.

```
interface Command {
    String name();          // "Move 3 objects", shown as "Undo Move 3 objects"
    void undo();
    void redo();            // also applies the edit the first time, when executed
    default boolean mergeWith(Command next) { return false; }
    default void discard() {}   // left the history for good: free what only it holds
}
```

## The history

Two stacks per history:

```
undo stack: [create, move, recolour]   <- top = the latest edit
redo stack: [delete]                   <- top = the next to redo
```

- **execute(c):** apply `c`, push it on the undo stack, clear the redo stack.
- **undo():** pop from undo, `undo()` it, push on redo.
- **redo():** pop from redo, `redo()` it, push on undo.
- A **new edit after some undos** throws away the redo stack (linear history; branching histories exist but are
  rare and confusing).
- A **limit** (a step count or a memory budget) drops the oldest commands; dropped commands get `discard()`.

### One history per document, or one for the app?

- **Per document** (Photoshop, most editors): undo in a document only touches that document. Simple and
  predictable.
- **Global** (some game editors): one history across everything. Needed when edits span documents (renaming an
  asset updates every document referring to it), but undo can then change a document you aren't looking at.

Start per document; add cross-document commands as a special case.

### The save point

The history remembers its depth when the document was saved. **Dirty = "not at the save point".** Undoing back
to it makes the document clean again. If the save point becomes unreachable (its state was in a dropped redo
branch, or a merge changed it), the document stays dirty until the next save.

## Writing commands

### Store values or store operations

| Kind | Undo by | Good for |
|---|---|---|
| **Values** (state-based) | writing back the stored "before" values | properties, transforms, small data; exact, never drifts |
| **Operations** | running the inverse operation | huge data (painting on a large image stores only the changed area) |

Store values by default. Copy mutable values (vectors, arrays) when storing them, so the stored "before" can't
change behind the command's back.

### Execute vs record

- **Execute:** the command makes the edit (delete, paste).
- **Record:** the edit was already applied live while the user dragged (moving with a handle, dragging a
  number). Show it live, and record **one** command when the drag ends, holding the values from before the drag
  and after it.

### Edit sessions (transactions)

Continuous edits become one step:

```
first change of a field  -> remember its value from before
...live changes...
the control is released  -> record one command (before -> after)
```

For tools that make many changes in one go:

```
history.beginGroup("Align objects");
  ...many commands...
history.endGroup();          // one composite command, undone as one step
```

### Merging

Typing "hello" shouldn't take five undos. `mergeWith(next)` lets the top command absorb a similar following one:
same target and property, within a short time window (0.5-1 s). A **seal** stops merging across boundaries that
must stay separate: an undo, a redo, a save, a change to a different property.

### Composite commands

A command made of commands, undone in **reverse order**: "paste 10 objects" is 10 adds, one step.

### Deleting without destroying

A deleted object can come back with undo, so delete **detaches** it (remove it from the document, keep the
instance) instead of destroying it. Undo re-attaches the same instance at the same position, so commands earlier
in the history that point to it still work. When the delete command is discarded for good, the object is freed.

### References: instances vs ids

Commands that hold direct references break when the objects are recreated (a reload, a restore from a
snapshot, network sync). Robust histories store **stable ids** and look the object up when undoing. Without stable
ids, the history has to be cleared whenever the document's objects are recreated.

## Rules

- **Every** change to a document goes through its history. A change that bypasses it can't be undone and breaks
  dirty tracking.
- Commands never call UI code. The UI reacts to the change ([Events](../runtime/03-events.md)).
- While an edit session is open, undo and redo either wait or close the session first; they never undo half an
  edit.
