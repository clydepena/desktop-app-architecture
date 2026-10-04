# Immediate, retained and declarative

The central architectural choice of a UI toolkit: **does the UI exist as objects between frames, and who keeps it in
sync with the data?**

## Immediate mode

The UI code **runs every frame** and describes the widgets as it goes. No widget objects persist.

```
if (ui.button("Save")) save();
ui.slider("Volume", ref volume, 0, 1);
if (ui.treeNode("Objects")) { for (o : objects) ui.text(o.name); ui.treePop(); }
```

**Good:**

- the UI always shows the current data: there is nothing to keep in sync
- trivial to write: ideal for tools, debug panels and inspectors
- conditional UI is just an `if`

**Hard:**

- **per-widget state** (is this tree node open? the scroll position? text being typed?) is stored behind the scenes,
  keyed by an **id** built from labels and nesting. Ids must be unique: two buttons both labeled "OK" in one window
  share state and confuse each other, so labels get hidden suffixes or explicit ids.
- knowing when an **edit ends** (to record one undo step for a drag, not one per frame) needs help from the toolkit
  ("was this item deactivated after an edit?").
- layout that depends on things not yet drawn this frame is usually **one frame late**.
- rich styling, animation and accessibility are harder, because there are no persistent objects to attach them to.

## Retained mode

A **tree of widget objects** persists. The application creates widgets, keeps references to them, updates their
properties and receives their events.

```
saveButton = new Button("Save"); saveButton.onClick(save);
volumeSlider = new Slider(0, 1); volumeSlider.onChange(v -> volume = v);
panel.add(saveButton); panel.add(volumeSlider);
...
volumeSlider.setValue(volume);   // must be called whenever volume changes elsewhere
```

**Good:** rich styling and animation, accessibility trees, layouts designers can build in visual tools, efficient
redrawing of only what changed.

**Hard:** the widgets hold a **copy** of the data, so every data change must be pushed into them. UIs that show stale
values because one path forgot to update a widget are the classic retained-mode bug; data binding exists to prevent
them ([State and data flow](03-state-and-data-flow.md)).

## Declarative (reactive) mode

The code describes **what the UI should look like for the current state**, like immediate mode, and the framework
**diffs** that description against a retained tree it manages, applying only the changes (**reconciliation**).

```
view(state) =
    Column(
        Button("Save", onClick = save),
        Slider(value = state.volume, onChange = setVolume))
```

When `state` changes, `view` runs again; the framework works out that only the slider's value changed. React,
SwiftUI, Flutter and Jetpack Compose work this way. It combines immediate mode's "no sync code" with retained mode's
persistent tree for animation, accessibility and efficient updates.

## Side by side

| | Immediate | Retained | Declarative |
|---|---|---|---|
| UI described | every frame, in code | once, then updated | whenever state changes, as a function of state |
| Widget objects | none (state stored by id) | a persistent tree | a persistent tree managed by the framework |
| Sync with data | automatic | manual, or bindings | automatic (re-describe, then diff) |
| Typical use | tools, debug UI, inspectors | classic desktop apps; game UIs built in designers | modern apps; increasingly games and tools |

The choice is less about speed (all three can be fast) than about **where state lives** and **how much sync code
people have to write**.
