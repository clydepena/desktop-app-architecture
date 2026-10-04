# Styling and theming

How widgets look, separately from what they are.

## Style separate from structure

What a button **is** (its structure and behavior) is kept apart from how it **looks**: colors, fonts, spacing,
borders, corner radius, shadows. Styles live in **style sheets** or style objects shared by many widgets (the
**Flyweight** pattern), so changing the look of every button is one change.

## Visual states

Widgets look different in each **state**: normal, hovered, pressed, focused, disabled, selected, checked, invalid.
Each state selects or modifies the style (the **State** pattern), often with a short transition between them.

## Themes

- **Dark and light** themes, following the **system theme** by default
  ([OS integration](../windowing/05-os-integration.md)), with a user override.
- **High-contrast** themes for accessibility.
- Themes change only style values, never structure.

## Design tokens

**Tokens** are named values used everywhere instead of raw numbers:

```
color.accent        = #3A7BFF
color.text.primary  = #F2F2F2
spacing.small       = 4
radius.control      = 4
font.body           = "Inter", 14
```

A theme is mostly a set of token values. Tokens keep a UI consistent and make themes, rebranding and accessibility
adjustments one-place changes.

## Skins for games

Game UIs are usually more decorative:

- **nine-slice images:** an image cut into nine parts, so corners stay sharp while edges and the center stretch to any
  size
- custom fonts, decorative frames, animated backgrounds
- styles per screen or per faction, built by artists in a UI designer

## Icons

Icons as **icon fonts** or **vector images** stay sharp at every scale; bitmap icons need several sizes
([Assets](12-assets.md)). Icons have accessible names, since screen readers can't describe a picture.

## Animation and transitions

Fades, slides, hover highlights, list items moving into place:

- run on **real time**, so menus animate while the game is paused ([Time domains](../time/04-time-domains.md))
- short, interruptible, and never blocking input
- reduced or removed when the user enables **reduced motion**

## Fonts and text styles

Text styles (body, heading, caption, monospace) are tokens too. The actual font for a piece of text comes from the text
style **and** the locale, since different scripts need different fonts
([Text rendering and layout](../localization/05-text-rendering-and-layout.md)).
