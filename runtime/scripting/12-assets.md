# Assets

Which assets scripting uses, and at which layer.

How source files are imported, cached and cooked into the runtime formats below is described in the
[asset pipeline](../../tool-framework/asset-pipeline/README.md).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Language runtime** | **script modules** | source code in development, bytecode in shipped builds |
| | **visual script graphs** | node graphs, state machines, event graphs |
| **Script API** | **API definition files** | generated type declarations and documentation for script editors |
| | **API version** | the version scripts and mods declare they need |
| **Isolation** | **sandbox / capability profiles** | what each kind of script may do (gameplay, UI, editor, mod) |
| | **resource limits** | memory, CPU and object limits per profile |
| **Mods** | **mod manifests** | name, version, required API version, dependencies on other mods, requested permissions, entry points |
| **Execution** | **script budgets and phase assignments** | per-frame budgets per category; which phase each kind of script runs in |

## Source and shipped forms

Scripts are edited as source and often **compiled to bytecode** for shipping: faster to load, slightly harder to
read, and checked for compile errors at build time instead of at the player's first run. Visual graphs are compiled
the same way.

## User data, settings and runtime state

- **User data, not assets:** user-written macros and personal automation scripts.
- **Settings:** mods enabled or disabled, the developer console on or off ([Settings](../04-settings.md)).
- **Runtime state:** contexts, script heaps, running coroutines. Script state that matters is saved as **domain data**
  through the data description, never as a dump of the script runtime's memory.

## Never in scripts

Secrets, API keys and credentials. Scripts are readable by anyone who has the files, including bytecode, which can be
decompiled.
