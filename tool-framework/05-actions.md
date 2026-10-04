# Actions

Everything the user can **trigger** is an action: Save, Undo, Duplicate, Toggle Grid, Open Settings. An action
is data plus a function:

```
record Action(
    String id,                  // "edit.duplicate" - stable, used by keybindings and plugins
    String label,               // "Duplicate"
    Icon icon,                  // optional
    String when,                // context rule: where it applies (below)
    Predicate<Context> enabled, // grayed out when false
    Consumer<Context> run
) {}
```

(The word "command" is used for both this and undo commands. Keep them apart: an action is something the user
triggers; an undo command is a reversible edit. Running an action often *creates* an undo command.)

## One list, many surfaces

Menus, toolbars, context menus, keyboard shortcuts and the command palette contain **no logic**. They are
views of the action registry:

```
menu "Edit"     -> [ "edit.undo", "edit.redo", "-", "edit.duplicate", "edit.delete" ]
toolbar         -> [ "doc.save", "preview.start", "preview.stop" ]
context menu    -> actions whose `when` matches the clicked item
command palette -> every action, searchable by label
```

Each surface asks the registry for the label, icon, shortcut text and enabled state. Adding an action makes it
available everywhere at once; renaming it renames it everywhere.

## Keybindings are data

```
[
  { "key": "ctrl+z",       "action": "edit.undo" },
  { "key": "ctrl+shift+z", "action": "edit.redo" },
  { "key": "delete",       "action": "edit.delete",     "when": "canvasFocus" },
  { "key": "f2",           "action": "item.rename",     "when": "outlineFocus" }
]
```

- **Defaults** ship with the app; the **user's** file overrides them ([Settings](../runtime/04-settings.md) layering).
- A keybinding editor lists actions with their keys and detects conflicts.
- Chords ("ctrl+k ctrl+s") are a sequence of keys mapped to one action.

## Context rules ("when")

The same key does different things in different places: Delete deletes a shape on the canvas, a character
in a text field, a file in the file browser. Each action declares where it applies, as a condition on **context
keys** maintained by the framework ([Selection and context](06-selection-and-context.md)):

```
"when": "canvasFocus && hasSelection && !isPreviewing"
```

Context keys are simple named values: `textInputFocus`, `canvasFocus`, `hasSelection`, `documentType == drawing`,
`isPreviewing`. Panels and tools set them; the action system reads them.

## Routing a key press

```
1. a text field has focus and wants the key?  -> the text field gets it (typing, its own undo)
2. find bindings for the key whose `when` matches the current context
3. several match? the most specific wins (focused panel before the window before global)
4. enabled? run it; otherwise do nothing (or beep)
```

This is the **chain of responsibility**: from the focused element outwards, the first that handles the key wins.

## Rules

- Actions read state through the context; they don't hold references to panels.
- Long actions start a job ([Jobs](../runtime/06-jobs.md)) instead of freezing the UI.
- Actions that change a document create undo commands; they never edit data directly.
- An action's id never changes once released: keybindings and plugins refer to it.
