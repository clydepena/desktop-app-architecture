# Data description

A generic way to **describe** data: which properties an object has, their types, ranges, labels and rules. The
description is written once, next to the data, and many systems read it instead of being hand-written per class:

- serialization ([Persistence](../tool-framework/03-persistence.md)): which fields are saved, under which names
- network synchronization: which fields are replicated
- scripting: which properties scripts can read and write
- diffing and copying values between objects
- editing tools ([Inspectors](../tool-framework/07-inspectors.md)): property panels, settings screens

The description belongs to the runtime because the shipped app needs it too (saving, networking, scripting); only
the inspectors that draw it are tool-specific. It carries **no UI code**.

## Three ways to describe properties

### 1. Reflection (automatic)

Read the class's fields at runtime and treat each according to its type: a number, a boolean, an enum, a vector,
a nested object. Nothing to write per class.

Limits: it can only guess intent. Is this `Vector4` a color? What range is valid? Should this field be saved,
shown, synchronized at all?

### 2. Annotations (metadata on fields)

The class labels its fields; the systems read the labels. The labels carry no code and no dependency on any tool,
so the data classes stay independent:

```
class Body {
    @Tooltip("Mass in kg") @Range(min = 0, max = 1000)
    float mass = 1;

    @ReadOnly Vector2 velocity;     // visible, not editable
    @Hide int internalFlags;        // saved, not shown
    @Color Vector4 tint;            // a color, not four numbers
    @Transient float cachedArea;    // not saved
}
```

A useful starting set: `Hide`, `Show` (show something not saved), `ReadOnly`, `Range`, `Step`, `Color`, `Label`,
`Tooltip`, `Header`, `Multiline`, `ShowIf`, `OnChange`, plus whatever serialization and replication need
(`Transient`, `Replicated`).

- A field can carry any number of different annotations; the same one twice needs the language's "repeatable"
  mechanism.
- The language doesn't check whether combinations make sense; the systems reading them decide and document
  precedence (`Hide` wins; `ReadOnly` ignores `Range`; `Color` on a non-vector is ignored with a warning).
- **Opt-out** (every saved field is described unless hidden) is convenient; **opt-in** (only marked fields) is
  explicit. Many tools use opt-out for content data and opt-in for anything security- or performance-sensitive
  (what a network peer or a script may touch).

### 3. Descriptors (code that lists properties)

For computed values, conditional rules and changes that must go through a setter, the class describes itself
through a small builder, still with no UI code:

```
void describe(PropertyList p) {
    p.floatProperty("Radius", () -> radius, v -> setRadius(v))
     .range(0, 10)
     .visibleIf(() -> shape == Shape.CIRCLE);
}
```

More code, full control. Use it only where annotations aren't enough.

## Localized text

Fields that hold user-visible text use a **localized text** type that stores a key, not the text itself. Because the
type is explicit, translation tools can extract every such field, inspectors can offer a key picker, and saved data
stays the same in every language. See [Strings and keys](localization/02-strings-and-keys.md).

## Reacting to outside changes

Whoever changes a described property from outside (an editor, a script, a network update, a loaded save) should
tell the owner, so it can react: rebuild a shape, mark a cache stale. Common forms: an `onChanged` hook on the
object, an `OnChange("method")` annotation, or the descriptor's setter.

## Practical rules

- **Cache the description per class.** Reading fields and annotations is slow compared with using the result;
  build it once per class and reuse it.
- **Read-only members** (`final` / `const`) are described as read-only, never written.
- **Language access rules** (module systems, private members) must let the reading systems in.
- **Version it with the data.** Renaming a described field breaks saved files and network peers unless the
  description keeps the old name as an alias.
