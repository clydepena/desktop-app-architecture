# Hot reload and state

Changing a script while the application runs, and seeing the result immediately, is much of the point of scripting.

## Reloading

```
the file changed (file watching)  ->  recompile  ->  swap the new code in  ->  next call runs the new version
```

File watching comes from the IO service ([Platform and file systems](../io/01-platform-and-file-systems.md)); scripts
are assets like any other, reloaded through their handles ([Resources](../07-resources.md)).

## Keeping the state

Swapping code is easy; keeping what the old code had built up is the hard part. From simplest to most capable:

| Approach | How it works | Good for |
|---|---|---|
| **Restart the script** | the new version starts fresh; state is lost | stateless rules, event handlers |
| **Serialize and restore** | save the script's state through the data description, reload, restore the fields that still exist, report the ones that don't | scripts with some state of their own |
| **Keep state in the host** | data lives in host components; scripts hold only behavior | the most robust design: reloading code never touches data |

The last option is a strong reason to keep scripts **mostly stateless**: positions, health and inventory live in host
data, which scripts read and change; the script itself is just logic.

## Failed reloads

A script with a syntax or compile error must not leave nothing running. The **old version keeps running**, and the
error is shown with its file and line until a working version is saved.

## Changes in shape

When a reloaded script's data no longer matches (a field renamed, a type changed), the reload reports exactly which
values couldn't be carried over, rather than silently resetting them.
