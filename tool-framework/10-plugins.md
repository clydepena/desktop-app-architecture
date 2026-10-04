# Plugins

The app defines **extension points**; plugins **contribute** to them. Done well, the core becomes a host and
most features are contributions, including the built-in ones (VS Code and Eclipse are built this way).

## Extension points

Each registry in the framework is an extension point:

| Extension point | A plugin contributes |
|---|---|
| Document types ([Documents](02-documents.md)) | a new kind of file the editor can open |
| Actions and keybindings ([Actions](05-actions.md)) | commands, menu entries, default keys |
| Views / panels | new tool windows |
| Resource loaders and savers ([Resources](../runtime/07-resources.md)) | support for new file formats |
| Property drawers ([Inspectors](07-inspectors.md)) | UI for new data types |
| Settings ([Settings](../runtime/04-settings.md)) | its own settings, shown in the settings screen |
| Importers / exporters | conversions to and from other formats |
| Tools | new editing tools (brushes, transform handles) |

## A plugin

```
interface Plugin {
    PluginInfo info();                     // id, name, version, required app version, dependencies
    void activate(PluginContext ctx);      // register contributions into the registries
    void deactivate();                     // optional: remove them (for unloading without restart)
}
```

`PluginContext` gives access to the registries and services the plugin is allowed to use, and records everything
the plugin registers, so it can all be removed at once (a scope, [Services and scopes](../runtime/02-services-and-scopes.md)).

## Two styles

- **Code plugins:** the plugin runs code in `activate`. Most flexible.
- **Declarative contributions:** a manifest lists actions, menus, keybindings, settings; code is loaded only when
  one is first used (lazy activation). Faster startup with many plugins.

Real systems mix both: the manifest declares, code implements.

## Rules

- **Plugins use public interfaces only.** Internal classes can change; plugins built against them break with
  every update.
- **Versioned API:** plugins declare which app/API version they need; incompatible ones are skipped with a message.
- **Isolate failures:** a plugin that throws during `activate` is disabled and reported, not allowed to stop
  startup ([Lifecycle](../runtime/01-lifecycle.md)).
- **Dependencies between plugins** are declared and activated in order.
- **Security:** plugins run with the app's permissions. Only load trusted plugins, or run untrusted ones in a
  separate process with a limited API.

## Start small

The first step needs no plugin loader at all: build the app's own features as contributions to registries
(document types, actions, loaders). External plugins can be added later without changing the core, because the
core already works through extension points.

## Plugins and scripts

Scripts are the other way to extend an application: written in a separate language, often untrusted and sandboxed,
reloaded constantly. How the two compare, and how editor scripts contribute to the same extension points, is covered in
[Editor scripting](../runtime/scripting/09-editor-scripting.md).
