# Visual scripting

Node graphs where nodes are actions or values and wires are execution order or data flow.

## What it is

- Graphs are **assets**, edited in a graph editor and saved like any document (with undo, dirty tracking and
  versioning).
- They are **compiled to the same bytecode** as text scripts, or interpreted node by node.
- They call the same **script API** ([Bindings](03-bindings.md)) as text scripts: a node is usually a bound function,
  generated from the same descriptions.

Common shapes: **event graphs** (when X happens, do Y), **state machines** (states and transitions), **data-flow
graphs** (values computed from other values: materials, procedural content).

## Strengths

- **Approachable:** no syntax to learn; available operations are listed and searchable.
- **Structure is visible:** what triggers what can be seen at a glance.
- **Live execution view:** highlighting nodes and wires as they run is an excellent debugging aid, hard to match in
  text.

## Weaknesses

- **Large graphs become unreadable**: the "spaghetti" is often literal.
- **Diffing and merging** in version control is hard, because graphs are stored as data files; two people editing the
  same graph usually conflict.
- **Performance** can be lower than text code, especially when interpreted node by node.
- **Refactoring tools** (rename everywhere, extract function) are less mature than for text.

## Remedies

- **Sub-graphs and functions:** collapse a group of nodes into one reusable node.
- **Comments and groups:** label regions of a graph, like comments in code.
- **Native nodes:** move heavy or complicated logic into host code, exposed as a single node.
- **Text-friendly storage and visual diff tools,** so changes can at least be reviewed.
- **Limits by convention:** if a graph needs scrolling in every direction, it should be split.
