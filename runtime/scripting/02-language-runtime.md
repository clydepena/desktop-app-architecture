# The language runtime

The part that actually runs script code.

## From interpreter to ahead-of-time compilation

| Approach | How it runs code | Speed | Notes |
|---|---|---|---|
| **Tree-walking interpreter** | walks the parsed source directly | slow | simplest to build |
| **Bytecode VM** | compiles once to compact instructions, then runs those in a loop | moderate | the usual choice for embedded languages (the **Bytecode** pattern) |
| **JIT** | compiles frequently run code to machine code while running | fast | complex; not allowed everywhere |
| **AOT** | compiles to machine code before shipping | fast | no runtime compilation; less dynamic |

Visual script graphs are usually **compiled to the same bytecode** as text scripts, or interpreted node by node.

## Platform limits

Some platforms (iOS and most game consoles) **forbid generating machine code at runtime**, so a JIT can't be used
there. Products that ship on them interpret bytecode, or compile scripts ahead of time as part of the build.

## A second heap

Most script runtimes have their own **garbage collector**. Consequences:

- the script heap is a memory category with its own budget ([Memory](../memory/README.md))
- its collection pauses add to the host's; the same advice applies inside scripts: avoid creating garbage every frame
- host objects held by scripts are kept alive (or not) by handles, not by the script collector
  ([Bindings](03-bindings.md))

## One context or many

A **context** is one isolated instance of the runtime: its own globals, heap and loaded modules.

| | One shared context | One context per mod or script group |
|---|---|---|
| Calls between scripts | fast and direct | through explicit messages or a shared API |
| Isolation | none: one script can break another | a broken or malicious mod can't touch others |
| Unloading | hard: references are everywhere | unload the whole context at once |
| Memory | lower | higher (one runtime instance per context) |

Trusted gameplay scripts often share one context; mods usually get one each.

## Threads

Most script runtimes are **single-threaded** per context: one thread runs one context at a time. Parallelism comes
from several contexts on several threads, or from host systems (written natively) doing the heavy work that scripts
request.
