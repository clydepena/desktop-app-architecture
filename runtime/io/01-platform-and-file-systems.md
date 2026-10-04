# Platform and file systems

The lowest layer: what the operating system offers for storage, behind one interface (the **Bridge / Adapter**
pattern).

## What the OS offers

- **Files:** open, read, write, seek, close; size and timestamps.
- **Directories:** list, create, delete, move, rename.
- **Memory mapping:** treat a file as memory; the OS loads pages as they are touched. Very fast for large,
  read-only data such as packed archives ([Packages](03-packages.md)). Writing through a mapping needs care (when is
  it on disk?), so it is mostly used for reading.
- **File watching:** notifications when files are created, changed, deleted or renamed: the basis of hot reload.
  Notifications arrive in bursts (an editor saving a file may write it several times), so they are **debounced**.
- **Permissions:** read-only files, folders the app may not write to, files locked by other programs.

## Well-known folders

Where an application is *allowed* to put things:

| Location | For | Example (Windows) |
|---|---|---|
| **install folder** | the shipped app and its content: **read-only** | `C:\Program Files\App\` |
| **user data** | settings, saves, recent lists, recovery copies | `%APPDATA%\App\` |
| **local / cache** | data that can be thrown away and rebuilt | `%LOCALAPPDATA%\App\Cache\` |
| **temp** | short-lived files | `%TEMP%` |
| **documents** | files the user creates and manages themselves | `Documents\` |

macOS and Linux have equivalents (`~/Library/Application Support`, `~/.config`, `~/.cache`, ...). The platform
layer asks the OS for these paths instead of hard-coding them.

Writing into the install folder is a classic bug: it fails without administrator rights, and updates wipe it.

## Platform differences to hide

| Difference | Why it matters |
|---|---|
| **Separators** (`\` vs `/`) | paths built by joining strings break on other platforms |
| **Case sensitivity** | `Texture.png` and `texture.png` are the same file on Windows and macOS by default, different files on Linux. Code that works on one fails on the other |
| **Long paths** | Windows limits path length unless long paths are enabled |
| **Unicode names** | file names in any language, and different normalization forms of the same text |
| **File locking** | Windows refuses to delete or rename a file that is open; other systems allow it |
| **Atomic rename** | renaming over an existing file is atomic on most local file systems, not on all network or cloud folders |
| **Sandboxes** | phones, consoles and app stores only allow certain folders, sometimes only through special APIs |

The rule for everything above the platform layer: **use one normalized path form** (forward slashes, no `..`,
consistent case) and let the platform layer translate.
