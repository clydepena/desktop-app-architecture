# Using the scripting service

How the rest of an application works with scripts: by exposing an API through bindings, attaching scripts to objects
and events, and running them inside the frame with budgets and error boundaries.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Modules** | load and compile scripts | `load(asset) -> ScriptModule`, `compile(source)`, errors with locations |
| **Calls** | run script functions | `call(module, "onStart", args)`, with a budget and error handling |
| **Bindings** | expose host types and functions | `expose(Type, description)`, `exposeFunction("spawn", fn)`, generated from the data description |
| **Instances** | attach scripts to objects | `attach(object, module) -> ScriptInstance` (a script component) |
| **Events** | connect host events to scripts | `on("playerEnteredZone", handler)`; scripts subscribe and emit |
| **Contexts and sandboxes** | isolate and limit | `createContext(capabilities, limits) -> ScriptContext`, one per mod |
| **Reload** | swap code while running | `reload(module)`, with state kept or reported |
| **Debugging** | tools | `setBreakpoint(...)`, attach a debugger, `profile(module)` |
| **Console** | run code live | `evaluate(context, "player.health = 100")` |

## How each part of an application uses it

| Part | How it uses scripting |
|---|---|
| **Data description** | generates the bindings: what scripts can read, write and call ([Data description](../08-data-description.md)) |
| **Time** | runs scripts in update phases; drives script coroutines and timers; budgets limit script time per frame |
| **Domain / gameplay** | exposes intentions and state; behaviors and rules live in script components and systems; the host keeps the data |
| **Resources and IO** | scripts are assets: loaded through the virtual file system, hot-reloaded on change, packaged as bytecode for shipping; sandboxed scripts see only their granted mounts |
| **Memory** | each context's heap is a budgeted category; native resources created by scripts are owned by scopes |
| **Rendering and audio** | reached only through the domain API ("set this sprite", "play this sound event"), never through GPU or mixer calls |
| **Input** | scripts read actions, never devices |
| **Networking** | server scripts have authority; client scripts are limited to presentation; script-callable RPCs are an explicit, validated list |
| **UI** | UI logic (menus, HUD behavior) reads data and calls actions |
| **Tool framework** | editor scripts go through actions and commands, contribute menu items, use selection and documents ([Editor scripting](09-editor-scripting.md)) |
| **Diagnostics** | script errors in the log and problems list with source locations; script timings in the profiler |
| **Settings** | users can disable mods or scripts; developer settings enable the console and debugger |
