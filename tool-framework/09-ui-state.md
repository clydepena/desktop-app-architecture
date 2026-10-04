# UI state

What the app remembers about **how it was being used**, as opposed to the user's work: window positions, panel
layout, open documents, scroll positions, the last folder in a file dialog, the active tool, expanded tree nodes.

## Keep it separate from documents

UI state is never part of a document and never makes one "unsaved". A user who only rearranged panels should not
be asked "save changes?". Opening a project on another machine should not move the other person's windows.

| State | Stored in | Saved |
|---|---|---|
| Window size and position, panel layout | user data folder | on exit, or debounced while changing |
| Open documents, active document, per-document view (camera, scroll, zoom) | per workspace, in the user data folder (not inside the shared workspace) | on close of the workspace |
| Last folders in file dialogs, recent searches | user data folder | when they change |
| Expanded/collapsed tree nodes, column widths | user data folder, keyed by view id | on exit |

## Session restore

On opening a workspace, restore the session: reopen the documents that were open, in the same tabs, with their
cameras and scroll positions. If a document no longer exists, skip it quietly (log it).

## Layout

- Layouts are data: which panels exist, where they are docked, their sizes. Ship a default layout; let users save
  named layouts ("Animation", "Review") and reset to default.
- Panels are identified by a stable id, so a saved layout survives a panel being renamed.
- A layout saved on a large monitor must still work on a small one: clamp windows to the visible screen area.

## Rules

- Save UI state with atomic writes too ([Persistence](03-persistence.md)): a corrupt layout file at startup is a bad first
  impression. If it can't be read, fall back to the default layout and log it.
- Version the format; on a version change, drop old UI state rather than migrating it. It's cheap to lose.
- Never block exit on saving UI state.
