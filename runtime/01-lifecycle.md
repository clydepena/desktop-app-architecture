# Lifecycle

The host: how the application starts, runs and stops. Most bugs in this area are **ordering** bugs (something
used before it exists, or after it was freed), so the main job of the lifecycle is to make the order explicit.

## Startup

A defined sequence, each step depending only on the ones before it:

```
1. platform        process, crash handler, working directory, command-line arguments
2. diagnostics     logging first, so everything after can report problems
3. settings        defaults, then the user's settings file
4. services        create the app-scoped services (see Services and scopes)
5. plugins         discover and load; they register their contributions
6. UI shell        main window, layout restored (see UI state)
7. session         open the workspace and documents from the command line or the last session
8. run             enter the main loop
```

See [Services and scopes](02-services-and-scopes.md), [Plugins](../tool-framework/10-plugins.md),
[Windowing and display](windowing/README.md) and [UI state](../tool-framework/09-ui-state.md).

Rules:

- **Fail early and clearly.** A missing GPU, an unreadable settings file or a corrupt plugin is reported at the
  step where it happens, with a fallback (default settings, plugin skipped) where one exists.
- **Show the window early.** If step 7 is slow (a big workspace), show the window with a progress indicator
  instead of a frozen splash screen.

## The main loop

The timing inside the loop (frame time, pacing, the fixed timestep, update phases, timers) is described in detail in
[Time and scheduling](time/README.md).

Two kinds:

| Kind | How it runs | Used by |
|---|---|---|
| **Event-driven** | sleeps until an event arrives (input, timer, file change), handles it, redraws | most desktop apps: text editors, office apps |
| **Continuous** | runs every frame whether or not anything happened | games, editors with a live viewport or animation |

Many editors switch between them: event-driven while idle, continuous while something animates or plays. This
saves battery and CPU.

One iteration, in a typical order:

```
poll platform events (input, window, OS messages)
run jobs that finished and handed results to the main thread  (Jobs)
deliver queued events                                          (Events)
update, in phases (input snapshot, fixed steps, update, late update)
draw: the scene and the UI
present
```

See [Jobs](06-jobs.md), [Events](03-events.md) and [Update phases and order](time/05-update-phases-and-order.md).

## Shutdown

The reverse of startup, preceded by a question:

```
1. ask      any document with unsaved changes? -> Save all / Don't save / Cancel  (Documents)
            (Cancel stops the shutdown here)
2. stop     cancel or finish background jobs; stop playback; wait for pending writes
3. save     session and UI state, settings that changed
4. close    documents, then the workspace, then plugins (each frees its own resources)
5. free     render outputs before their windows; app services in reverse creation order;
            GPU resources before the GPU device
6. exit
```

See [Documents](../tool-framework/02-documents.md), [UI state](../tool-framework/09-ui-state.md) and
[Windowing and rendering](windowing/06-windowing-and-rendering.md).

Rules:

- **A close request is a request.** The window's close button, the OS logging off and "Quit" all go through
  step 1, and step 1 can cancel.
- **Free in reverse order of creation.** Anything created after X may depend on X.

## Other host responsibilities

- **Single instance:** launching the app a second time hands the arguments (a file to open) to the running
  instance instead of starting another.
- **Command-line mode:** the same app without a UI (convert, build, export, run tests). Possible only if the
  domain doesn't depend on the UI shell.
- **Crash handling:** catch fatal errors, write a report and the recovery data ([Persistence](../tool-framework/03-persistence.md)), then exit.

## Patterns

- **Template method:** the host defines the sequence; an application fills in hooks (`onStart`, `onUpdate`,
  `onClose`).
- **Ordered initialization / reverse teardown.**
