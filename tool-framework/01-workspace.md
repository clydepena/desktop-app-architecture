# Workspace

The container that groups documents and settings: a project folder, a VS Code workspace, a solution in an IDE, a
single `.blend` file in Blender. Most editors have exactly one open at a time.

## What a workspace owns

- **A root**, usually a folder. Paths inside the workspace are stored **relative to it**, so the workspace can
  be moved or shared.
- **A workspace file** describing it: name, format version, settings, the starting or main document.
- **Workspace settings:** a layer in the settings stack ([Settings](../runtime/04-settings.md)), shared by everyone who opens it.
- **Workspace-scoped services:** resource caches, indexes, file watchers ([Services and scopes](../runtime/02-services-and-scopes.md)). They are
  created when it opens and freed when it closes.

```
interface Workspace {
    Path root();
    String name();
    Settings settings();          // the workspace layer
    Scope scope();                // services living as long as the workspace
}
```

## Operations

| Operation | What happens |
|---|---|
| **Create** | make the folder and the workspace file from a template; open it |
| **Open** | read the workspace file (check its version), create its scope, restore its session |
| **Close** | ask about unsaved documents, save the session, close the documents, dispose the scope |
| **Switch** | close, then open (never two half-open workspaces) |

Closing must free **everything** the workspace created. A classic bug: something registered globally when the
workspace opened and never removed on close, so the next workspace collides with the leftovers.

## The recent list

- Stored in the **user's** data folder, not inside any workspace.
- Each entry: path to the workspace file, display name, last opened time.
- Missing entries stay in the list, marked "not found", with **Locate...** (pick the new path) and **Remove**.
- Opening a workspace moves it to the top; the list is capped (20 or so).

## The launcher (optional)

A start screen shown when no workspace is open: recent workspaces, Create, Open. Two common shapes:

- **Same window, two modes** (launcher mode, then editor mode): one UI context, simpler.
- **Separate launcher process** that starts the editor: isolation, but two programs to maintain.

## Relative vs absolute paths

- Paths **inside** the workspace: relative to the root, with forward slashes, in every file.
- Paths **outside** it (a shared asset library, an SDK): absolute, or relative to a named location the user
  configures ("library:", "sdk:"), never machine-specific paths written into shared files.
