# Coupling: what may depend on the pipeline

The direction rule: the pipeline is a **tool** system. It depends on the runtime (IO, jobs, resources, the data
description) and on each service's **import and cook libraries**. **Nothing in the runtime depends on the pipeline**:
the shipped product contains loaders for cooked data, and no importers at all.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Pipeline -> IO, jobs, resources, data description** | runtime services below it |
| **Pipeline -> each service's importers and cookers** | registered into it, like plugins |
| **Asset browser, inspectors, editors -> pipeline** | the tool layer uses it |
| **Build machines, command-line cook tools -> pipeline** | the same pipeline, without a UI |

## Keep decoupled

| System | How it relates instead |
|---|---|
| **Runtime loaders** | read only cooked formats; the shipped product never needs PSD, FBX or WAV parsers |
| **Domain / gameplay** | refers to assets by id or handle; never knows about importers, sidecars or caches |
| **Runtime services** | ship their runtime half (loaders); their import and cook half lives in tool modules |
| **The editor's UI** | the pipeline never depends on it, so it runs headless on build machines |
| **Documents** | stay the user's work; derived data is never treated as a document |

## Mistakes to avoid

- **Committing derived data** or the cache to version control.
- **Timestamp-based rebuild decisions:** too much rebuilt after a branch switch, too little when a timestamp is wrong.
- **Leaving the importer version out of cache keys**, so fixes never reach old results.
- **Non-deterministic importers** (timestamps, random ids, hash-map order in output), which defeat caching and
  reproducible builds.
- **Path-based references**, so renames break content.
- **Importing on the main thread:** the editor freezes every time a file is saved in another tool.
- **Shipping importers and source formats** in the product, or **cooking everything** instead of what is reachable.
- **Editor-only data left in shipped builds.**
- **Deleting assets without checking who references them.**
- **Sources committed without their sidecars**, giving the same asset different ids on different machines.

## Testing the boundary

- **Determinism:** importing or cooking the same inputs twice gives byte-identical results.
- **Clean vs incremental:** a build from an empty cache equals an incremental build after the same changes.
- **Golden assets:** a small set of test sources with known expected outputs, checked whenever an importer or cooker
  changes.
- **Validation on every commit:** build machines cook for every platform, and new problems fail the build.
