# Editor scripting

Automating the editor itself: batch-renaming assets, running an import pipeline, placing objects procedurally, adding
custom menus, panels and validation checks. This part of scripting builds on the
[tool framework](../../tool-framework/).

## Through actions and commands, never around them

A script that changes a document does so through the same **undo commands** a person's edits use
([Undo](../../tool-framework/04-undo.md)). Its changes are then:

- undoable
- marked as unsaved, through the history's save point
- compatible with collaboration, where commands are also the messages sent to others

A script that changes data directly breaks all three.

## One script run, one undo step

By default, everything a script run changes is grouped into one composite command, named after the script ("Undo
'Align selected objects'"). The user undoes a script's work in one step, not five hundred.

## Scripts can contribute actions

A script registers a menu item, a toolbar button or a shortcut through the action registry
([Actions](../../tool-framework/05-actions.md)), with an "enabled when" rule like any other action. To the user, it looks
like a built-in feature.

## Selection and documents

Editor scripts read the selection and the active document through the same services panels use
([Selection and context](../../tool-framework/06-selection-and-context.md)), so "do this to the selected objects" works
the same in every script.

## Scripts vs plugins

| | Plugins | Editor scripts |
|---|---|---|
| Written in | the host language, compiled | the scripting language |
| Trust | installed deliberately, full access | often project-local; sandboxed or "trust this project" |
| Lifetime | loaded at startup | run on demand, or reloaded constantly |
| Good for | new document types, importers, deep integrations | automation, small tools, project-specific helpers |

Both contribute to the same extension points (actions, panels, validators); see [Plugins](../../tool-framework/10-plugins.md).
