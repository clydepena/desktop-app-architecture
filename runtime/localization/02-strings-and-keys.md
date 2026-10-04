# Strings and keys

The core rule of internationalization: **no user-visible text in code or data, only keys.**

```
ui.label(text("menu.newGame"))                               // "New Game", "Nouvelle partie", "Neues Spiel"
dialog.show(text("quest.foundItems", count = 3, item = itemName))
```

## String tables

A **string table** maps keys to text, one table per locale. The **source language** table is written by the team; the
others are translations of it. Code and data hold only keys; the text is looked up at the moment it is shown.

## Never build sentences by concatenation

```
"You have " + n + " items"       // can't be translated
```

Word order, plurals, grammatical case and gender change around the number in other languages. Translators need
**whole sentences with placeholders**, so they can arrange them correctly ([Message formatting](03-message-formatting.md)):

```
"inventory.count": "You have {count, plural, one {# item} other {# items}}."
```

The same applies to sentences assembled from fragments ("Delete" + " " + "file"), and to text glued onto numbers or
names in the UI.

## Keys are stable

A key is how every translation finds its source. Renaming a key orphans its translations, so renames are migrations
done by tools ([The translation pipeline](08-translation-pipeline.md)), never by hand.

Readable keys (`menu.newGame`) help everyone; generated ids work too, as long as tools show the source text next to
them.

## Context for translators

The same English word can need different translations:

- "Open" as a verb on a button, and "Open" as a shop's status
- "Save" as an action, and "Save" as a noun (a saved game)
- "Fire" as a command, and "Fire" as a hazard

These need **different keys**, even though the English is identical. Each key also carries **context** for translators:
a comment ("button on the main menu, max 20 characters"), who is speaking, and ideally a screenshot.

## Localized fields in data

Text in data (item names, quest descriptions, tooltips) is localized too. In the
[data description](../08-data-description.md), such fields have a **localized text** type that holds a key, not a plain
string:

```
class Item {
    LocalizedText displayName;     // holds "item.healthPotion.name"
    LocalizedText description;
    int price;
}
```

Because the type is explicit, extraction tools find every localizable field automatically, the inspector shows a key
picker with a preview instead of a text box, and saved data stores the key.
