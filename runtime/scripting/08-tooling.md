# Tooling

Scripting is only as good as its tools. Without them, scripts become the least debuggable part of an application.

## Errors with locations

Every script error shows the **file, line and the script's own call stack**, and appears in the problems list
([Diagnostics](../05-diagnostics.md)) where clicking it opens the file at that line. Host stack traces alone (deep
inside the interpreter) are useless to script authors.

## A debugger

Breakpoints, stepping, inspecting variables, watching expressions, for scripts running inside the application. Many
script runtimes support the **Debug Adapter Protocol**, so existing code editors can attach to the running application
instead of a custom debugger being written. Visual scripts get the live execution view described in
[Visual scripting](07-visual-scripting.md).

## A profiler

Time and allocations per script and per function, shown next to the host's systems in the same frame timeline, so
"the frame is slow" can be traced to "this script's update takes 4 ms".

## A console

A live prompt that runs a line of script against the running application:

```
> player.health = 100
> spawn("enemy", player.position + vec2(5, 0))
```

Invaluable for testing, debugging and live tuning. It runs with the console's capabilities, so it is usually available
only in development builds or behind a developer setting.

## Autocomplete and documentation

Generated from the bindings ([Bindings](03-bindings.md)):

- **type declaration files** for code editors, so they offer completion and catch typos before running
- **API reference pages**, listing every exposed type, property, function and event with its description

Because they are generated, they stay correct as the API grows.
