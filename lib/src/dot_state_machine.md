# Dot State Machine
---

## What `.` can be in Urdu text

**1. Decimal number** — `3.14`, `0.5`
Keep the dot. Pattern: `digit . digit`

**2. URL or domain** — `urdu.com`, `www.urdu.com`
Keep the dot. Pattern: `latin_alphanum . latin_alphanum`

**3. File extension** — `file.txt`, `data.csv`
Same pattern as URL, indistinguishable without context.

**4. Abbreviation** — `Rs.`, `Dr.`, `Mr.`
This one is hard. The dot follows Latin letters but is not between two letter sequences in a predictable way. `Rs. 500` has a dot followed by a space then a digit.

**5. Urdu ordinal marker** — `۱.` `۲.` `۳.`
In your own test file, lines start with `۱.` `۲.` etc. Here the dot follows an Urdu digit and is followed by a space. This dot should be **stripped** — it is punctuation marking a list item, not a decimal.

**6. Latin sentence terminator** — `sentence.` at end
Should be stripped.

**7. Ellipsis-style repetition** — `...`
Already handled — ELLIPSIS `U+2026` is in punctuation set. But three literal dots `...` are three separate FULL_STOP_LATIN characters and are not currently caught as a group.

---

## The Core Ambiguity

The difference between cases 1 and 5 is:

```
3.14    →  3.14     digit DOT digit        → keep
۱. آج   →  ۱ آج     digit DOT space/letter → strip
urdu.com → urdu.com  alpha DOT alpha        → keep
Rs. 500 →  Rs 500   alpha DOT space digit  → strip
```

The dot-between-two-digits rule alone is not enough. You also need to know what follows the dot — specifically whether the character **after** the dot is a digit or an alpha character, or a space.

---

## What the state machine needs to track

```
State 1: idle
State 2: digitFound       — last cp was a digit
State 3: latinAlphaFound  — last cp was a Latin letter
State 4: dotFound         — dot seen after digit or Latin alpha, waiting to see what follows
```

On seeing the next character after the dot:

| Previous | Dot followed by | Action |
|---|---|---|
| digit | digit | re-emit dot, stay in digit flow |
| digit | space / letter / end | strip dot |
| Latin alpha | Latin alpha | re-emit dot (URL/abbreviation base) |
| Latin alpha | space / digit / end | strip dot |
| Urdu digit | anything | strip dot (ordinal marker) |

---

## The question before you build it

Do you want to handle **all** these cases now, or start with just `digit . digit` (decimal numbers) the same way you handled `digit : digit`?

The decimal case is clean and follows exactly the same pattern as colon. The URL/Latin-alpha case introduces `isLatinAlphaFound` as a new flag and makes the state machine meaningfully more complex. It also sets a precedent — once you handle URLs you will want to handle `www.` prefixes, which pulls you toward full URL detection.

My suggestion: build `digit . digit` now, exactly like colon. Add a comment marking the Latin-alpha case as a known gap to be revisited in Phase 1 when token types are introduced. That keeps the state machine honest about what it does without overreaching.

What do you want to do?