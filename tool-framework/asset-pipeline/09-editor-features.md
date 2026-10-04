# Editor features

What people see of the pipeline inside the editor.

## The asset browser

- folders, and **search** by name, type, label and content (tags, dimensions, duration)
- **thumbnails** and previews, generated in the background ([Importers](03-importers.md))
- **filters** by type, label, problems, or "modified since..."
- **drag and drop** from the browser into scenes, documents and inspector fields

It is built with the UI toolkit ([UI toolkit](../../runtime/ui/README.md)), with virtualized grids and lists so projects
with tens of thousands of assets stay fast.

## Importing new files

Files dropped from the operating system, or copied into the project folder, are discovered and imported with presets.
For types without a preset, an **import settings** dialog appears before the first import.

## Reference-safe operations

Because content refers to ids ([The asset database](02-asset-database.md)):

- **rename and move** keep ids, so nothing that refers to the asset breaks
- **delete** first checks who refers to the asset: "used by these 12 assets", with **cancel**, **delete anyway**
  (leaving missing references, reported as problems) or **replace with...** another asset
- **duplicate** creates a new id for the copy, and optionally for its sub-assets

All of these are undoable operations through the history ([Undo](../04-undo.md)).

## References and dependencies

- **"Find references"**: everything that uses this asset
- **"Dependencies"**: everything this asset uses
- a **dependency viewer** showing the graph around an asset, to understand why changing one file rebuilds others, or
  why an asset ships in a build ([Cooking](07-cooking.md))

## Missing references

References to ids that no longer exist (an asset deleted, a sub-asset renamed in its source tool) are listed in the
problems list, with "find" to locate them and "replace" to point them at another asset in one step.

## Bulk operations

Reimport, reset settings, apply a preset, or compare settings across a selection of assets. Bulk changes to import
settings are one undo step, and trigger one batch of reimports.
