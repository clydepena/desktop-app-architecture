# Background import

The editor never waits for the pipeline.

## From a saved file to an updated view

```
someone saves hero.png in an image editor
  -> file watching reports the change (debounced)         IO
  -> the pipeline schedules a reimport job                 jobs, with a priority
  -> the importer runs on a worker thread
  -> the result is stored in the cache
  -> the resource handle for hero.png swaps to the new data
  -> every viewport, inspector and preview using it updates
```

File watching comes from [IO](../../runtime/io/01-platform-and-file-systems.md); the swap happens behind the asset's
handle ([Resources](../../runtime/07-resources.md)), the **Observer** pattern end to end.

## Priorities

Assets the user is looking at come first:

- assets visible in a viewport or open in an editor: **high**
- assets in the open level or document: **normal**
- everything else, thumbnails, previews: **low**

A progress indicator shows the queue ("Importing 12 of 340"), and individual imports can be canceled.

## Placeholders while importing

An asset being imported for the first time shows a **placeholder** (a checkerboard, a gray box, silence), so the
editor stays usable. An asset being **re**imported keeps showing its previous version until the new one is ready.

## Bulk changes

Switching branches or pulling a large update changes thousands of files at once. Importing them one by one, in the
order the file watcher reports them, wastes work: an asset may be imported before its dependencies and again after.
Instead, changes are **collected**, then imported once, as a batch, in dependency order
([Dependencies and incremental builds](05-dependencies-and-incremental-builds.md)), mostly from the cache if the
other branch was built before.

## Failures

A failed import keeps the previous good result (or the placeholder) in place, reports the problem on the asset, and
retries when the file changes again. The editor never ends up without data because one file was briefly half-written
by the tool that saved it.
