# Inspectors

Property panels generated from the data description ([Data description](../runtime/08-data-description.md)): one
generic panel edits any described object, so editors don't hand-write a UI per class. The same panels serve
settings screens ([Settings](../runtime/04-settings.md)).

## Drawers: how a type is shown

The editor keeps a **registry of drawers**, one per type, with a fallback for unknown types:

```
drawers.register(float.class,   FloatDrawer);           // drag field, or a slider if it has a Range
drawers.register(boolean.class, CheckboxDrawer);
drawers.register(Color.class,   ColorDrawer);
drawers.register(Handle.class,  ResourcePickerDrawer);  // thumbnail + "pick another"
drawers.register(Curve.class,   CurveEditorDrawer);
```

- The drawer reads the description's metadata: a `Range` turns a drag field into a slider, `ReadOnly` grays it
  out, `Label` and `Tooltip` set the text, `Header` adds a section title.
- Fully custom UI for one class (a timeline for an animation) is a drawer registered for that class, living in
  the editor, not in the class.
- Drawers live in the tool layer, so the data classes never depend on them.

## Editing through the inspector

- **Every edit is an undo command** (target, property, before, after); continuous drags are one edit session
  ([Undo](04-undo.md)).
- **Notify the owner** after an edit (the description's `onChanged` hook or setter), so it can react.
- **Multi-object editing:** show a property once for all selected objects; mixed values show as "-"; an edit
  applies to all, as one undo step.
- **Read-only modes** (a preview running, a locked document) gray out the whole panel instead of hiding it.

## Inspecting any object

Because the description is generic, one inspector can edit anything: objects in a document, workspace settings,
user preferences, resource import settings. Limits to handle:

1. **Unknown types** need a drawer; the fallback shows them read-only.
2. **How deep to follow references:** only into allowed classes (a list, or an `Inspectable` marker), never into
   library or platform internals.
3. **Cycles** (A -> B -> A): track the objects being drawn, plus a depth limit.
4. **Shared objects:** editing one instance referenced from two places changes both; the inspector can't tell.
5. **Collections** need add / remove / reorder controls, each an undo command.
6. **Speed:** use the cached description, never re-read fields and annotations every frame.
