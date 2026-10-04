# Coupling: what may depend on IO

IO is one of the lowest runtime services: **almost everyone may depend on its interface**. IO itself depends only
on the platform layer, the jobs system and logging; it never knows about assets, documents, the renderer or the
game. Anything flowing back (completions, file-watch events) is delivered to the caller's worker or the main thread.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Lifecycle** | creates the VFS and the mounts: the single owner of mount setup |
| **Resources, persistence, settings, diagnostics, plugins** | read and write through logical paths and the scheduler |
| **Audio streaming workers** | issue range reads; the audio thread itself never does IO |
| **Networking downloads** | stream into `cache:` or staging locations |
| **Editor browsers, importers, packaging tools** | the tool layer, above IO |

## Keep decoupled

| System | How it gets what it needs instead |
|---|---|
| **Domain / gameplay** | goes through persistence (saves) or resources (assets); never opens files or builds paths from strings |
| **Rendering, the audio thread, input** | receive decoded data; never touch storage |
| **Undo, actions, selection** | work on in-memory documents; IO is only involved when persistence saves |
| **Saved data** | stores logical paths or stable ids, never absolute physical paths |

## Mistakes to avoid

- **Blocking reads or writes on the main thread**, including "just a small config file": on a busy network drive it
  can take seconds.
- **Absolute or machine-specific paths** in saved files, shared settings or code.
- **Hard-coded separators**, and **case-sensitivity bugs** that pass on Windows and fail on Linux.
- **Writing into the install folder**, or leaving caches and temp files with no cleanup.
- **Thousands of tiny loose files** in shipped builds instead of packages.
- **Reading huge files whole** instead of streaming or memory mapping them.
- **Trusting file contents:** downloaded files, mods and user files are **untrusted input**, validated like network
  data (sizes, counts, paths that try to escape their mount).
- **Leaving files open:** they stay locked on Windows and leak handles.
- **Ignoring write failures:** silent data loss.

## Testing the boundary

- **An in-memory backend** (the **Null Object** / fake idea): mount `mem:` in place of real folders, and tests run
  fast, in parallel, with no disk and nothing to clean up.
- **Fault injection:** a backend that fails on purpose (disk full, permission denied, a truncated file, a write that
  fails half-way) to test error messages and atomic saves.
- **Latency simulation:** an artificially slow backend reveals hidden blocking reads and missing loading states.
