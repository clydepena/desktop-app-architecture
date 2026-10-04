# Bindings

Bindings are what scripts see of the host: the objects, functions and events they may use. They are the **coupling
point** between scripts and everything else, so they get the most design care.

## Handles, never raw objects

A script may keep a reference to a game object for minutes. Meanwhile, the host may destroy it, reload it or move it.
If the script held a raw object or pointer, using it afterwards would crash or corrupt memory.

So script-side objects are **proxies** holding **handles with generations** (the same idea as
[GPU resources](../rendering/02-gpu-resources.md)):

```
script: door.open()
binding: look up door's handle -> still valid?  yes -> call the host
                                               no  -> raise "this object no longer exists" in the script
```

The host can destroy, reload or move objects freely; scripts get a clear error instead of a crash.

## Generated from the data description

The properties scripts may read and write come from the same [data description](../08-data-description.md) that drives
saving, inspectors and replication. Methods meant for scripts are **marked explicitly** (a `ScriptCallable`
annotation, or a list), never "everything public":

```
class Door {
    @Scriptable float openSpeed;           // readable and writable from scripts
    @ScriptCallable void open() { ... }    // callable from scripts
    void rebuildCollision() { ... }        // internal: not visible to scripts
}
```

Generating bindings keeps them in sync with the code, and produces the documentation and type declarations for
script editors at the same time ([Tooling](08-tooling.md)).

## Two directions

- **Host -> script:** the host calls script functions when things happen: `onStart`, `onUpdate`, `onCollision`,
  `onClick`, custom events.
- **Script -> host:** scripts call the API: spawn, move, play a sound event, read input actions, set a timer, request
  an action.

## A small, high-level API

Scripts express **intentions** ("play the footstep event", "request pickup", "show this dialog"), not low-level
operations ("allocate a voice", "write to a buffer", "create a GPU texture"). A small API is:

- easier to **secure** (fewer ways to do damage)
- easier to **document** and learn
- easier to keep **compatible** across versions, which mods depend on

## Crossing the boundary has a cost

Every call converts values between the host's types and the script's (marshaling): numbers, strings, arrays, objects.
Chatty APIs (thousands of tiny calls per frame) are slow. Prefer:

- calls that do more per call ("move all these objects")
- data passed in bulk (arrays) instead of item by item
- heavy loops implemented in the host, called once by the script

## Versioning

The script API has a **version**. Additions are compatible; removals and changes are not. Breaking changes are rare,
announced, and come with a migration path (old names kept as aliases for a while), because every mod written against
the old API breaks at once otherwise.

## Patterns

- **Facade:** one script API in front of many host systems.
- **Adapter:** converting between the two type systems.
- **Proxy:** script objects standing in for host objects.
