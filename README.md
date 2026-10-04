# Desktop application and editor architecture

Under the UI and the subject matter of any serious desktop application or editor sits the same set of systems:
rendering, audio, input, files, background jobs, settings, documents, undo, actions, selection and more.
Photoshop, VS Code, Blender, Unity and a word processor all have roughly the same machinery underneath. These
documents describe that machinery, one system at a time: what each one is for, how it is usually built, and the
mistakes to avoid.

Code is pseudo-code in a Java/C#-like style; nothing depends on a particular language or UI toolkit.

> **About these documents:** they are compiled from research into how existing desktop applications and editors
> are built, and were written with the help of AI. Treat them as learning material: an overview of common
> approaches and trade-offs, not an authoritative specification. Verify details against primary sources before
> relying on them.
>
> **Corrections welcome:** if you find an error, an outdated detail or something unclear, please open an issue or
> a pull request.

## The layers

```
+--------------------------------------------------------------+
| Application    the UI shell, or the shipped product itself    |  specific to each application
+--------------------------------------------------------------+
| Domain         what the app is about: text buffers, layers    |  specific to each application
|                of an image, 3D scenes...                      |
+--------------------------------------------------------------+
| Tool framework documents, undo, actions, selection,          |  editors and tools only
|                workspace, inspectors, clipboard, UI state,    |
|                asset pipeline                                 |
+--------------------------------------------------------------+
| Runtime        lifecycle, services, events, settings, jobs,   |  shared by the editor and
| services       resources, rendering, audio, input, IO, time,  |  the shipped app
|                memory, threads, scripting, localization;      |
|                UI toolkit at the top                          |
+--------------------------------------------------------------+
| Platform       window, GPU API, audio device, file system,    |  thin wrappers that hide
| abstraction    sockets, threads, clocks, input devices        |  the OS differences
+--------------------------------------------------------------+
| OS, drivers, hardware                                          |
+--------------------------------------------------------------+
```

**The dependency rule:** each layer depends only on the layers below it.

- The runtime never knows what a paragraph, a layer or a 3D model is.
- The domain never knows which UI toolkit draws it.
- The tool framework is used by editors; the shipped app can leave it out entirely.
- Only the platform layer knows which operating system or GPU API it runs on.

**Which layer does a system belong to?** Ask whether the shipped product needs it. Undo and documents: only while
editing, so they are tool framework. Rendering, audio and settings: the product needs them too, so they are
runtime services. Talking to the OS directly: platform.

The payoff: one runtime hosts any number of editors and the product itself; the domain can run without any UI
(a command-line converter, a server, automated tests); and porting to another OS or GPU API touches only the
platform layer.

## The documents

### [platform/](platform/)

Thin, OS-specific wrappers: windows and surfaces, the GPU API, audio devices, the file system, sockets, threads,
clocks, input devices, virtual memory. Each service's platform part is documented inside that service's folder (the
GPU API in [rendering](runtime/rendering/01-rhi.md), sockets in [networking](runtime/networking/01-platform-and-transport.md),
and so on); the [folder's README](platform/README.md) links them all.

### [runtime/](runtime/)

| # | System | One-line job |
|---|---|---|
| 1 | [Lifecycle](runtime/01-lifecycle.md) | start, run, shut down, in a defined order |
| 2 | [Services and scopes](runtime/02-services-and-scopes.md) | how systems find each other, and how long they live |
| 3 | [Events](runtime/03-events.md) | announcing changes without depending on the listeners |
| 4 | [Settings](runtime/04-settings.md) | layered, typed configuration |
| 5 | [Diagnostics and feedback](runtime/05-diagnostics.md) | logging, notifications, problems, crash reports |
| 6 | [Jobs](runtime/06-jobs.md) | background work with progress, cancellation, main-thread handoff |
| 7 | [Resources](runtime/07-resources.md) | loading external data by reference: handles, caches, ids |
| 8 | [Data description](runtime/08-data-description.md) | describing data so saving, scripting, sync and tools can use it |

Larger services have their own folder, with a reading series inside:

| Service | Folder | One-line job |
|---|---|---|
| Rendering | [runtime/rendering/](runtime/rendering/README.md) | turning descriptions of what is visible into frames: RHI, GPU resources, render proxies, the frame pipeline, assets, and how the rest of an app uses and stays decoupled from it |
| Audio | [runtime/audio/](runtime/audio/README.md) | turning descriptions of what should be heard into a continuous signal: device layer, clips, emitters and voices, the mixer graph, real-time threading, sound events, assets, and coupling |
| Networking | [runtime/networking/](runtime/networking/README.md) | sharing state and calling services across machines: transport, messages, sessions, replication and RPC, prediction, request/response services, collaboration, security, assets, and coupling |
| Input | [runtime/input/](runtime/input/README.md) | turning device signals into intentions and routing them to the right place: devices, players, routing and focus, action mapping, timing, text input, feedback and prompts, accessibility, assets, and coupling |
| IO | [runtime/io/](runtime/io/README.md) | making storage simple, uniform and safe: file systems, the virtual file system, packages, the IO scheduler and streaming, streams and formats, safe writing, assets, and coupling |
| Time and scheduling | [runtime/time/](runtime/time/README.md) | clocks, the frame loop and pacing, the fixed timestep, time domains, update order, timers and coroutines, time slicing, determinism, assets, and coupling |
| Windowing and display | [runtime/windowing/](runtime/windowing/README.md) | a place on the screen: the message pump, windows and events, displays and scale, several windows, OS integration, and how windowing and rendering meet without depending on each other |
| Memory | [runtime/memory/](runtime/memory/README.md) | where everything lives: virtual memory, lifetimes, allocation strategies, garbage-collected languages, ownership, layout, budgets and pressure, GPU memory, assets, and coupling |
| Threads and synchronization | [runtime/threads/](runtime/threads/README.md) | doing several things at once safely: platform threads, locks, atomics and memory ordering, handing data between threads, threading models, task systems, hazards, debugging, assets, and coupling |
| Scripting | [runtime/scripting/](runtime/scripting/README.md) | changing behavior without rebuilding: kinds of scripting, runtimes, bindings, execution, sandboxes, hot reload, visual scripting, tooling, editor scripting, assets, and coupling |
| Localization | [runtime/localization/](runtime/localization/README.md) | every language and region: locales, keys instead of text, plurals and formatting, data formats, text rendering and layout, localized assets, switching, the translation pipeline, tools, assets, and coupling |
| UI toolkit | [runtime/ui/](runtime/ui/README.md) | interactive views of the application's state: native vs custom-drawn, immediate vs retained vs declarative, data flow, layout, focus, styling, rendering, accessibility, tool vs product UIs, assets, and coupling |

### [tool-framework/](tool-framework/)

| # | System | One-line job |
|---|---|---|
| 1 | [Workspace](tool-framework/01-workspace.md) | the project: root folder, project settings, recent list |
| 2 | [Documents](tool-framework/02-documents.md) | data + file + unsaved state + history; open/save/close |
| 3 | [Persistence](tool-framework/03-persistence.md) | files and back: formats, versions, atomic save, autosave, recovery |
| 4 | [Undo](tool-framework/04-undo.md) | every change is a reversible command |
| 5 | [Actions](tool-framework/05-actions.md) | everything the user can trigger, with shortcuts and context rules |
| 6 | [Selection and context](tool-framework/06-selection-and-context.md) | what the user is working on right now |
| 7 | [Inspectors](tool-framework/07-inspectors.md) | property panels generated from the data description |
| 8 | [Clipboard and drag-and-drop](tool-framework/08-clipboard-and-drag-drop.md) | typed data moving between places |
| 9 | [UI state](tool-framework/09-ui-state.md) | remembering layout and session, separate from documents |
| 10 | [Plugins](tool-framework/10-plugins.md) | extension points and contributions |

A larger system has its own folder, with a reading series inside:

| System | Folder | One-line job |
|---|---|---|
| Asset pipeline | [tool-framework/asset-pipeline/](tool-framework/asset-pipeline/README.md) | turning source files into runtime data: the asset database, importers, the derived data cache, incremental builds, cooking, packaging and collaboration |

## How they fit together

A typical user action in an editor, end to end:

```
key press
  -> keybindings map it to an action id                 (actions)
  -> context rules pick the action for the focused view (selection and context)
  -> the action reads the selection, builds a command   (actions, selection)
  -> the document's history executes the command        (undo)
  -> the document becomes dirty                         (documents)
  -> a "document changed" event is published            (events)
  -> views redraw; the title shows "*"                  (application: UI shell)
  -> later, Save writes the file atomically             (persistence)
```

None of those steps needs to know what the document contains.

## Reading order

For a first pass: [Lifecycle](runtime/01-lifecycle.md), [Services and scopes](runtime/02-services-and-scopes.md),
[Documents](tool-framework/02-documents.md), [Undo](tool-framework/04-undo.md),
[Actions](tool-framework/05-actions.md). The rest of the framework builds on those five.

Then the services, in roughly this order:

1. [Time and scheduling](runtime/time/README.md), [Memory](runtime/memory/README.md) and
   [Threads and synchronization](runtime/threads/README.md): what every other service
   runs on.
2. [Rendering](runtime/rendering/README.md): the clearest example of the shape every service shares (a platform
   layer, proxies, a narrow interface, the direction rule). [Audio](runtime/audio/README.md),
   [Networking](runtime/networking/README.md) and [Input](runtime/input/README.md) then read quickly, as variations
   on it.
3. [IO](runtime/io/README.md) and [Windowing and display](runtime/windowing/README.md): what the services stand on
   and present into.
4. [Scripting](runtime/scripting/README.md), [Localization](runtime/localization/README.md) and the
   [UI toolkit](runtime/ui/README.md): the services closest to the people using and extending the application.

Every service folder ends with the same three pages: how to use it, what may depend on it, and the assets it uses.

## Common patterns, and where they appear

| Pattern | Used by |
|---|---|
| Command | undo, actions, network messages and RPCs, input snapshots, editor scripts |
| Memento | undo (state snapshots), autosave, play/preview snapshots, rollback, saved window placement |
| Composite | grouped undo steps, hierarchical documents, menus, the UI widget tree, the clock hierarchy, audio buses |
| Observer / publish-subscribe | events, observable properties and bindings, settings and locale changes, memory pressure |
| Registry | document types, actions, resource loaders, importers and cookers, plugins, inspector drawers |
| Strategy | loaders/savers by file type, editor tools, allocators, layout algorithms, pacing policies, render paths |
| Proxy | resource handles, render proxies, replicated objects, remote procedure calls, script objects, logical paths |
| Flyweight | one shared instance per resource, shared materials and clips, shared styles |
| Chain of responsibility | input and UI event routing, action context rules, locale fallback chains, VFS overlays |
| Bridge / Adapter | every platform layer (GPU, audio, sockets, files, windows, input), UI backends, script bindings |
| Mediator | the application shell joining windowing and rendering |
| Decorator | stream pipelines, tracking allocators, pseudo-localization |
| Object pool / arena | GPU resources, audio voices, network messages, per-frame memory |
| Producer / consumer | render, audio, network and IO threads handing work to and from the main thread; task queues |
| Interpreter / Bytecode | scripting, message formats |
| Pipes and filters | the asset pipeline (import, cook, package), stream pipelines |
| Memoization | the derived data cache, layout caches |
| Game Loop / Update Method | the frame loop and its update phases |
| Null object | null renderer, offline network, in-memory files, manual clock, headless UI: testing every boundary |
| Dependency injection / service locator | services and scopes |
| Template method | lifecycle hooks |
| Unit of work | saving every changed document together |
| Data transfer object | file formats kept separate from live objects |

## License

These documents are dedicated to the public domain under [CC0 1.0 Universal](LICENSE): you may copy, modify,
distribute and use them for any purpose, including commercially, without asking permission and without giving
credit.
