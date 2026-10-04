# Settings

Configuration that the user (or a team) changes: theme, font size, grid size, autosave interval, keybindings,
default paths.

## Layers

Each layer overrides the one below it:

```
document     (rare: per-file overrides, e.g. indentation in a text file)
workspace    shared with everyone who opens the project, stored in the workspace
user         this person, on this machine, stored in the user's data folder
defaults     shipped with the app, never written
```

Reading a setting returns the value from the highest layer that has one. Writing goes to one chosen layer
(usually user). A settings screen shows where each value comes from and offers "reset to default".

## Typed settings with a schema

Each setting is declared once, with its type, default and metadata:

```
Setting<Float>  GRID_SIZE = setting("editor.gridSize", 0.25f)
                              .range(0.01f, 10f).label("Grid size").category("Editor");
Setting<String> THEME     = setting("ui.theme", "dark").choices("dark", "light", "classic");
```

From the schema, the framework gets:

- validation (a value outside the range, or of the wrong type, falls back to the default with a warning)
- a generated settings screen (the same inspectors that edit objects: [Inspectors](../tool-framework/07-inspectors.md))
- search ("grid" finds the grid settings)
- documentation

## Storage

- Text files (JSON, TOML), one per layer, holding only values that differ from the defaults. A new default in a
  new version then reaches users who never changed that setting.
- Unknown keys are kept, not deleted (a newer version or a plugin may own them).
- Writes are atomic ([Persistence](../tool-framework/03-persistence.md)).

## Change notification

Settings publish a change event per key. Systems react live (the theme changes immediately) or read the value
when they next need it. Settings that can only apply on restart say so in the UI.

## Settings vs UI state vs documents

| | Settings | UI state | Documents |
|---|---|---|---|
| Changed by | the user, deliberately | as a side effect of using the app | the user, as their work |
| Example | theme, grid size | panel layout, last opened folder | the drawing, the text |
| "Unsaved changes?" | no, saved immediately | no, saved on exit | yes |
| Shared with a team | workspace layer, yes | no | yes |

Mixing these up is a common source of noise: layout changes showing up as unsaved document changes, or personal
preferences committed to a shared project.
