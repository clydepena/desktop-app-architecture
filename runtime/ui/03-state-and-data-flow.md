# State and data flow

What the UI shows must have **one source of truth**. Most UI bugs are two copies of the same state disagreeing.

## Two kinds of state

| Kind | Examples | Lives in |
|---|---|---|
| **Application state** | documents, settings, the selection, the player's health | the application; never only in a widget |
| **Widget-local UI state** | scroll position, which tree nodes are open, text typed but not yet committed, hover | the widget; saved as [UI state](../../tool-framework/09-ui-state.md) when it matters |

Widgets **show** application state and **request** changes to it; they don't own it.

## Patterns that keep the UI separate

| Pattern | Idea | Strength |
|---|---|---|
| **MVC** (Model-View-Controller) | the model holds data, the view shows it, the controller turns input into model changes | the classic separation |
| **MVP** (Model-View-Presenter) | a presenter drives a passive view through a small interface | the presenter can be tested without any UI |
| **MVVM** (Model-View-ViewModel) | a view model exposes **observable properties and commands** shaped for the view; the view **binds** to them | little glue code; the view model is testable without UI |
| **Unidirectional data flow** (Elm, Redux-style) | state -> view; UI events -> actions -> a function computes the new state -> view again | one direction: easy to trace, replay and debug |

## Data binding

A **binding** connects a widget property to a data property, so a change on one side updates the other:

```
bind(volumeSlider.value, settingsViewModel.musicVolume)        // two-way
bind(healthBar.fraction, playerViewModel.healthFraction)       // one-way, data -> view
```

Bindings rely on **observable** data (the **Observer** pattern): the data announces changes, and bound widgets update.
They remove most retained-mode sync code. Declarative toolkits get the same effect by re-describing the UI from state.

## View models

A **view model** shapes application data for one screen: "health as a fraction from 0 to 1", "the inventory as a sorted
list of display items", "Save is enabled when the document is dirty". It sits between the domain and the UI:

- the domain doesn't know about widgets
- the UI doesn't reach into the domain's internals
- the view model can be tested without either

## In editors, edits go through commands

A widget that changes a document doesn't write the data directly. It triggers an **action** or an **undo command**
([Undo](../../tool-framework/04-undo.md)), so the change is undoable, marks the document unsaved and works with
collaboration. Continuous edits (dragging a slider) are recorded as one command when the edit ends: the inspector's
**edit sessions** ([Inspectors](../../tool-framework/07-inspectors.md)).

In a game, the equivalent is **intentions**: a menu button requests "start new game" or "equip item 12"; gameplay code
decides what happens.
