# Message formatting

Messages are templates with placeholders, plural rules and selections, filled in when shown:

```
"quest.foundItems": "{count, plural, one {You found # {item}.} other {You found # {item}s.}}"
```

## Named placeholders

Placeholders have **names** (`{playerName}`, `{count}`), never positions (`%s`, `{0}`). Translators can then move them
anywhere in the sentence, which they often must: word order differs between languages.

## Plurals in every language

Plurals aren't "one vs many". The CLDR defines up to six **plural categories** (zero, one, two, few, many, other), and
each language uses some of them:

| Language | Forms |
|---|---|
| Japanese, Chinese, Korean | 1 (no plural forms) |
| English, German | 2 (one, other) |
| French, Spanish, Italian | one, other, plus "many" for large numbers such as a million (recent CLDR versions); in French, 0 also counts as "one" |
| Russian, Ukrainian | 3 (one, few, many) plus "other" for fractions |
| Polish | 3 plus "other" |
| Arabic | 6 |

`if (n == 1)` in code is a bug in most languages. The rule for a number comes from the **locale**, and the message
provides a variant for each category that language uses.

## Gender and agreement

"{player} picked up {their} sword" changes form with the player's gender in many languages; adjectives and articles
agree with the gender and number of nouns. Message formats support a **select** on a value:

```
"pickup": "{gender, select, female {Elle a pris l'épée.} male {Il a pris l'épée.} other {Iel a pris l'épée.}}"
```

The value comes from data (the character's gender, the item's grammatical gender in that language), never from
guessing.

## Message format standards

Two widely used formats handle placeholders, plurals and selects in one syntax that translators and tools understand:

- **ICU MessageFormat:** supported by most locale libraries and translation tools.
- **Fluent:** designed for natural-sounding translations, letting translators add variants the source language
  doesn't need.

Both are small languages interpreted at runtime (the **Interpreter** pattern), with the rules per locale supplied by
locale data (the **Strategy** pattern).

## Rich text

Messages often contain formatting ("Press **{key}** to continue", a link inside a sentence). Markup is kept inside the
message as tags the translator can move, rather than splitting the sentence into separately translated pieces.
