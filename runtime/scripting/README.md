# Scripting

How an application lets people change what it does **without rebuilding it**: designers writing gameplay rules,
modders adding content, users automating their tools, numbers tuned while the application runs.

Scripting is the most **open** service. Everything else is written by the application's programmers; scripts are
written by others, sometimes strangers. It has the familiar shape (a platform layer, a runtime, a narrow interface,
the same dependency direction), and its special problems all sit at the **boundary between the host and the
scripts**:

- **the boundary itself:** what scripts can see and call, and what happens when a script holds an object the host has
  destroyed
- **safety:** a mod is untrusted code running on the user's machine
- **performance:** interpreted code, and a second garbage-collected heap
- **change:** hot reload without losing state, and an API that mods depend on and that can't break casually

It spans two layers: **runtime scripting** ships with the product (gameplay, mods, UI logic); **editor scripting**
automates the tools and builds on the tool framework (actions, commands, undo).

## The layers inside a scripting service

```
+---------------------------------------------------------------------+
| Consumers          gameplay scripts, UI scripts, editor and tool       |  written by others
|                    scripts, mods, console commands, automation          |
+---------------------------------------------------------------------+
| Script API         the objects, functions and events scripts may use;  |  the contract
| (bindings)         generated from the data description; versioned       |
+---------------------------------------------------------------------+
| Execution          when scripts run (phases, events, coroutines),      |
|                    budgets, errors, determinism                         |
+---------------------------------------------------------------------+
| Isolation and      sandboxes, capabilities, resource limits,           |  safety
| security           one context per mod                                  |
+---------------------------------------------------------------------+
| Language runtime   interpreter / bytecode VM / JIT for a text language, |
|                    or a visual graph compiler; its own garbage collector |
+---------------------------------------------------------------------+
| Platform           memory, threads, and whether JIT is allowed           |  platform layer
+---------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Kinds of scripting](01-kinds-of-scripting.md) | text languages, visual scripting, WebAssembly, hot-reloaded native code, expressions |
| 2 | [The language runtime](02-language-runtime.md) | interpreters, bytecode, JIT and AOT, platform limits, the script heap, contexts |
| 3 | [Bindings](03-bindings.md) | the boundary: handles not raw objects, generated bindings, two directions, cost |
| 4 | [Execution](04-execution.md) | where scripts attach, when they run, coroutines, budgets, errors, determinism |
| 5 | [Isolation and security](05-isolation-and-security.md) | sandboxes, capabilities, limits, contexts per mod, trust |
| 6 | [Hot reload and state](06-hot-reload-and-state.md) | swapping code while running, keeping state, failed reloads |
| 7 | [Visual scripting](07-visual-scripting.md) | node graphs: strengths, weaknesses, remedies |
| 8 | [Tooling](08-tooling.md) | errors, debuggers, profilers, consoles, generated documentation |
| 9 | [Editor scripting](09-editor-scripting.md) | automating the tools through actions and undo; scripts vs plugins |
| 10 | [Using the scripting service](10-using-the-scripting-service.md) | the public interface, and how each part of an app uses it |
| 11 | [Coupling: what may depend on scripting](11-coupling.md) | safe and unsafe dependencies, fake hosts, sandbox and compatibility tests |
| 12 | [Assets](12-assets.md) | modules, graphs, API definitions, sandbox profiles, mod manifests |

## How scripting compares

| | Data description | Plugins | Scripting |
|---|---|---|---|
| Describes or adds | **what data is** | features in the host's language | **behavior, in a separate language or graph** |
| Written by | programmers | programmers, extension authors | designers, modders, users, programmers |
| Trust | n/a | installed deliberately, full access | **often untrusted: sandboxed** |
| Changes while running | no | rarely | **constantly: hot reload** |
| Boundary | annotations and descriptors | extension points | **bindings, handles, budgets, sandboxes** |

## Patterns in scripting

| Pattern | Where |
|---|---|
| Interpreter, Bytecode | running script code (both in *Game Programming Patterns*) |
| Facade | the script API in front of many host systems |
| Adapter | bindings converting between the two type systems |
| Proxy | script-side objects holding handles to host objects |
| Component | script components attached to objects |
| Observer | host events that scripts subscribe to |
| Command | editor scripts changing documents through undo commands |
| Coroutine | sequential script logic spread over frames |
| Sandbox / object capabilities | scripts can only use what they were given |
| Strategy | interchangeable languages or runtimes |
| Null object | null host services for headless script tests |

## Related

- [Time and scheduling](../time/README.md): update phases, coroutines, budgets.
- [Memory](../memory/README.md): the script heap as a budgeted category.
- [Localization](../localization/README.md): every language and region: keys, formatting, text layout and localized assets.
- [Data description](../08-data-description.md): the source the bindings are generated from.
- [Plugins](../../tool-framework/10-plugins.md): the other way to extend an application.
- [Undo](../../tool-framework/04-undo.md), [Actions](../../tool-framework/05-actions.md): what editor scripts go through.
