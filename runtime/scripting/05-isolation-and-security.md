# Isolation and security

Mods and user scripts are **untrusted code running on the user's machine**. Without limits, a mod can delete files,
steal data or install malware.

## The sandbox

Scripts get **no direct access** to files, the network, processes or the operating system. Everything they need goes
through host APIs, which check permissions. Many language runtimes include dangerous built-ins by default (file IO,
loading native libraries, running arbitrary code from strings); those are removed or replaced in sandboxed contexts.

## Capabilities

A script receives only the abilities it was **granted**, as objects it can call (the **object-capability** idea: you
can only use what you were given):

- read its own mod folder (a read-only mount in the [virtual file system](../io/02-virtual-file-system.md))
- write to a save slot
- no network access unless explicitly allowed
- no access to other mods' data

Permissions are declared in the mod's **manifest** ([Assets](12-assets.md)) and can be shown to the user before
installing or enabling it.

## Resource limits

- **memory:** the script heap's budget ([Memory](../memory/README.md))
- **CPU:** instruction or time limits ([Execution](04-execution.md))
- **objects:** how many things a script may create (objects, timers, sounds)

## One context per mod

Each mod runs in its own context ([The language runtime](02-language-runtime.md)), so mods can't reach into each other
or into the host's internals, and a misbehaving mod can be unloaded on its own.

## WebAssembly

WebAssembly is attractive for untrusted code because its sandbox is strict **by design**: a module can only touch its
own memory and the functions the host explicitly imports into it.

## Prefer data mods

Mods that only add **data** (new levels, textures, sounds, tuned numbers) are far safer than code mods. Offering rich
data modding reduces how much code modding is needed.

## Trusted, but not blindly

Editor scripts written by the team are trusted. But project scripts from a **downloaded** project run with the user's
permissions too. Editors warn before running scripts from an unfamiliar project ("trust this project?"), as many code
editors do for folders.

## Networked games

Clients can't be trusted to run scripts honestly. Gameplay scripts with consequences run on the **server**; client
scripts are limited to presentation (effects, UI). RPCs that scripts may call are an explicit list, validated like any
other client message ([Security](../networking/08-security.md)).
