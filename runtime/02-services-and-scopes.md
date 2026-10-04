# Services and scopes

How systems find each other without global variables, and how long each one lives.

## The problem with singletons

`Something.getInstance()` callable from anywhere is convenient, but:

- **Hidden dependencies:** a class's real needs aren't visible in its constructor.
- **Wrong lifetime:** a singleton lives as long as the app, but much of what it holds belongs to one workspace or
  one document. Closing a workspace then leaves stale state behind, which the next workspace trips over.
- **Hard to test or run twice:** two documents, two windows or a test need separate instances.

## Services

A service is one system with one job, used through an interface: the document manager, the action registry,
the resource cache, the job system, the logger, the settings.

```
interface ServiceRegistry {
    <T> T get(Class<T> type);                 // throws if missing: a missing service is a startup bug
    <T> void register(Class<T> type, T service);
}
```

Two ways to hand services to the code that needs them:

- **Dependency injection:** pass them in, usually through the constructor. Dependencies are visible; tests pass
  fakes. Preferred for core systems.
- **Service locator:** code asks the registry. Simpler to wire, dependencies less visible. Fine for UI panels and
  plugins.

Either way, there is no `static` access.

## Scopes

Services live in **scopes**, nested by lifetime:

```
app scope            settings, logging, action registry, keybindings, job system
  workspace scope    resource caches, file watchers, indexes, workspace settings
    document scope   the document's history, selection, view state
```

- A scope can **see its parents'** services; a parent never sees its children's.
- **Closing a scope disposes everything created in it**, in reverse order of creation. Closing a workspace frees
  its caches and watchers automatically, with no cleanup list to forget.
- A lookup walks outwards: a document asks for the resource cache, finds it in the workspace scope.
- Scopes are also where **memory lifetimes** come from: a scope can own an arena or pools, freed all at once when
  it closes ([Lifetimes](memory/02-lifetimes.md)).

```
class Scope implements AutoCloseable {
    Scope parent;
    <T> T get(Class<T> type);        // here, else parent.get(type)
    <T> T add(Class<T> type, T s);   // owned: closed with this scope if it's Closeable
    void close();                    // closes children first, then own services in reverse order
}
```

## Context objects

A middle ground often seen in smaller editors: one "context" object passed to every panel, holding the shared
services as fields. Fine at small sizes. Signs it has outgrown that:

- it holds things with different lifetimes (app, workspace, document) and gets fields set to null on close
- everything depends on it, so any change touches everything

At that point, split it by scope.

## Rules

- Services are created during startup ([Lifecycle](01-lifecycle.md)) or when their scope opens, never lazily from random places.
- A service depends only on services in its own scope or a parent scope.
- Disposing is the scope's job: services don't free each other.
