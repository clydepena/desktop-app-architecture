# Selection and context

"What is the user working on right now?" Actions need it to know what to act on; panels need it to stay in sync.

## Three levels

```
active document      which document is in front (the focused tab or window)
focused view         which panel has keyboard focus (canvas, outline, inspector, text field)
selection            what is selected inside the active document (objects, text range, pixels)
```

## The selection model

One selection object **per document**, shared by every view of that document:

```
interface Selection<T> {
    List<T> items();
    T primary();                          // the last clicked: what single-item tools act on
    void set(Collection<T> items);
    void add(...); void remove(...); void toggle(...); void clear();
    void apply(Mode mode, Collection<T> items);   // REPLACE / ADD / TOGGLE / SUBTRACT (no key / shift / ctrl / alt)
    Event<SelectionChanged> changed();
}
```

- Views don't talk to each other. The outline and the canvas both read and change the same selection, and
  redraw when it changes. That is why they stay in sync.
- **Primary item:** with several items selected, single-item tools (the inspector, a transform handle's pivot) use the last
  one clicked.
- **Removed items** (deleted, closed) are dropped from the selection automatically.
- **Selection is not saved with the document** (it's UI state, [UI state](09-ui-state.md)), and changing it is usually not an
  undo step. Some apps do record it (so undo restores what was selected); if so, merge it into the edit's
  command rather than adding separate steps.

## Context keys

The context is a set of named values describing the current situation, used by action rules ([Actions](05-actions.md)):

```
context.set("canvasFocus", true);
context.set("hasSelection", !selection.isEmpty());
context.set("documentType", "drawing");
context.set("isPreviewing", false);
```

Who sets what:

| Key | Set by |
|---|---|
| `xxxFocus` | each panel, when it gains or loses focus |
| `textInputFocus` | the UI toolkit, while a text field is being edited |
| `hasSelection`, `selectionCount` | the selection of the active document |
| `documentType`, `isDirty` | the active document |
| mode keys (`isPreviewing`, `toolActive`) | the systems that own those modes |

Contexts nest: a panel's context inherits its window's, which inherits the app's. A lookup walks outwards,
which is how the focused panel's value wins.

## Modes

Some states change what is allowed everywhere: a preview or run mode, a modal tool in progress, a
read-only document. Model them as context keys plus checks at the **entry points** (actions refuse to run,
editing views become read-only), not as scattered `if` statements inside each feature.
