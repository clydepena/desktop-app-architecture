# Coupling: what may depend on scripting

The direction rule: **scripts depend on the script API; the script API (the bindings) depends on the systems it
exposes; those systems don't depend on scripting.** They work the same whether a script, a person or a test calls
them. The scripting service itself depends on runtime services below it: memory, time, IO, the data description,
diagnostics.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Bindings -> domain, data description, actions** | exposing them is the bindings' job; like a bridge, the one place that knows both sides |
| **Scripting service -> memory, time, IO, diagnostics** | runtime services below it |
| **Editor scripts -> actions, commands, documents** | through the same doorways people use |
| **Debuggers, consoles, profilers** | the tool layer, above scripting |

## Keep decoupled

| System | How it relates to scripts instead |
|---|---|
| **Domain systems** | never call scripts by name or depend on a script existing; they raise events, and scripts subscribe |
| **Rendering, audio, networking internals** | not exposed; scripts reach them only through domain intentions |
| **Undo, documents** | never bypassed: editor scripts change documents through commands |
| **Host internals** | no raw objects or pointers cross the boundary, only handles and copies |

## Mistakes to avoid

- **Exposing raw host objects:** scripts crash the host by using destroyed objects, and every internal becomes a
  public API that can never change.
- **An API that grows to "everything public":** impossible to secure, document or keep compatible.
- **Script errors that crash or freeze the host:** no error boundary, no budget.
- **Heavy per-frame work in scripts,** or chatty calls across the boundary every frame.
- **Unsandboxed mods:** file deletion, data theft, malware.
- **A JIT on platforms that forbid it.**
- **Hot reload silently losing state,** or a failed reload leaving nothing running.
- **Nondeterministic script features** inside deterministic simulations.
- **Editor scripts changing documents directly,** bypassing undo and dirty tracking.
- **Breaking the script API between versions** without a version number or migration path: every mod breaks at once.

## Testing the boundary

- **Headless script tests:** run scripts against the domain with no rendering, windows or audio (null services).
- **A fake host API:** test scripts against mock bindings, and test bindings without scripts.
- **Budget and error tests:** endless loops, deep recursion and thrown errors must stop the script, not the host.
- **Sandbox tests:** scripts trying to reach files, the network or other mods must be refused.
- **API compatibility tests:** a collection of old mods and scripts that must still load and run on new versions.
