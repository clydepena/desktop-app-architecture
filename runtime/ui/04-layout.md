# Layout

Deciding where every widget goes and how big it is, and doing it again whenever anything changes: the window size, the
scale, the language, the content.

## Measure, then arrange

Most layout systems work in two passes:

1. **Measure:** each widget reports how much space it would like, given the space available (a label: the size of its
   text; a list: the size of its rows).
2. **Arrange:** each parent gives each child a rectangle, using its layout algorithm.

## Containers

Each container implements one layout algorithm (the **Strategy** pattern):

| Container | Arranges children |
|---|---|
| **Stack / box** | in a row or a column |
| **Flex** | in a row or column, growing and shrinking to share the space by weights |
| **Grid** | in rows and columns |
| **Anchors / constraints** | attached to edges, centered, or positioned relative to siblings |
| **Dock / splitters** | editor-style layouts with resizable regions and tabs |
| **Scroll view** | content larger than its box, with scroll bars |
| **Absolute** | at fixed positions (rarely right; fine for overlays) |

## Sizing

Each widget's size along each axis is one of:

- **fixed** (a 24-unit icon)
- **fit content** (a button as wide as its label)
- **fill** (take the remaining space, shared by weights)

with **minimum and maximum** limits.

## Units and scale

Layout uses **logical units**, converted to pixels with the window's content scale
([Displays, scale and modes](../windowing/03-displays-scale-and-modes.md)), so a button is the same physical size on
every monitor. A user **UI scale** setting multiplies on top of that.

## Built for localization

- Boxes **grow** for longer translations; text **wraps** where it can; fixed widths are the exception.
- Layouts use **start and end** instead of left and right, so right-to-left languages mirror automatically.
- Truncation with an ellipsis is the last resort, with the full text on hover.

See [Text rendering and layout](../localization/05-text-rendering-and-layout.md).

## Virtualization

A list of 100,000 rows can't create 100,000 widgets. **Virtualized** lists, trees and grids create widgets only for the
**visible** rows, and reuse them as the user scrolls, so cost depends on what is on screen, not on how much data there
is. Asset browsers, log panels, hierarchies and tables in tools depend on it. The rows not yet created are stand-ins in
the **virtual proxy** sense: they exist as data until they are needed.

## Invalidation and caching

Recomputing the layout of the whole UI on every change is wasteful. Widgets mark themselves **dirty** when their size or
content changes, and only the affected branch is measured and arranged again.

## Safe areas

Some screens have regions where UI must not go: TV **overscan** (edges cut off by some televisions), phone **notches**
and rounded corners, areas covered by system UI. Layouts respect a **safe area** reported by the platform.
