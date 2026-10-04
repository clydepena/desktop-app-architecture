# Diagnostics and feedback

Telling the user (and the developer) what happened, at the right level of interruption.

## Logging

```
log.debug / info / warn / error(message, optional exception)
```

- **Sinks:** where messages go: a console panel in the app, a rotating file in the user's data folder, the
  terminal during development. Each sink has its own minimum level.
- **Categories** ("resources", "documents", "plugin:xyz") so a panel can filter.
- **Thread-safe:** messages from workers are queued and shown on the main thread ([Jobs](06-jobs.md)).
- **Capture stray output:** a library printing to standard output/error should still reach the log.
- Log the reason with the failure: "can't save poster.drawing: access denied", never just "save failed".

## Levels of interruption

Choose the least disruptive one that still gets the message across:

| Level | Blocks? | Use for |
|---|---|---|
| Log entry | no | details, history, developer information |
| Status bar text | no | "Saved poster.drawing" |
| Notification (toast) | no, disappears or waits in a list | finished jobs, warnings needing attention later |
| Inline message | no | a form's validation ("name already exists") next to the field |
| Modal dialog | yes | decisions that must be made now: unsaved changes, overwrite a file |

Modal dialogs for information ("Saved!") train users to click without reading. Reserve them for decisions.

## Problems list

Errors and warnings tied to a place, as in an IDE:

```
[error]   poster.drawing   layer "Logo"   missing image "images/logo.png"
[warning] icons.atlas                     region 12 extends past the image
```

- Each entry links to its location: clicking selects the object or opens the file.
- Entries are owned by whoever produced them (a validator, an importer) and replaced when that producer runs
  again, so the list is never stale.

## Crash reports

On a fatal error: write the stack trace, the app version, the platform, and the last log lines to a file; save
recovery copies of dirty documents ([Persistence](../tool-framework/03-persistence.md)); then show, on the next start, "the app closed unexpectedly"
with the report and the recovery offer.

## Developer diagnostics

Optional panels for the developer: frame time, memory, loaded resources and their states, running jobs, event
counts. Built on the same services, so they cost nothing when closed.
