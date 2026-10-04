# Kinds of scripting

"Scripting" covers several different ways of letting people add behavior without rebuilding the application.

| Kind | Examples | Good for | Watch out for |
|---|---|---|---|
| **Embedded text language** | Lua, JavaScript, Python, a .NET language, a custom language | gameplay logic, mods, tool automation | binding work, a second runtime, performance |
| **Visual scripting** | node graphs, state machines, event graphs | designers and artists who don't write code | hard to diff and merge in version control; large graphs become unreadable |
| **WebAssembly** | modules compiled from many languages | a portable, strongly **sandboxed** target for untrusted code | binding data in and out is more manual |
| **Hot-reloaded native code** | the application's own language, recompiled while it runs | fast iteration for programmers, full speed | no sandbox; a crash takes down the host |
| **Data with expressions** | formulas in tables or configuration ("damage = base x level") | designers tuning numbers | grows into a bad language if pushed too far |

## Choosing

- **Who writes it?** Designers and artists favor visual scripting or a simple language; programmers favor a real
  language or native code; modders need something documented, stable and safe.
- **Is it trusted?** Code from the team can be trusted; code from strangers needs a strong sandbox
  ([Isolation and security](05-isolation-and-security.md)), which favors WebAssembly or a carefully restricted
  language runtime.
- **How fast must it run?** Per-frame logic over thousands of objects belongs in native code or a JIT; event handlers
  and rules run fine interpreted.
- **Which platforms?** Some forbid generating machine code at runtime ([The language runtime](02-language-runtime.md)).

## Two kinds at once

Many products use two: a visual or simple language for designers, and native or WebAssembly code for heavier logic,
both calling the same host API. Keeping one **script API** ([Bindings](03-bindings.md)) behind both means everything
written in one can be reached from the other.

## Expressions deserve a warning

Small expression languages in data ("cost = 10 + level * 5") are useful. They also tend to grow: conditions, then
loops, then functions, until they are a full language without a debugger, an error model or documentation. When that
happens, it is usually better to embed a real language than to keep extending the expression syntax.
