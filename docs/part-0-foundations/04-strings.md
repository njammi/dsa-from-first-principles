# Strings (Arrays of Characters, and Why They're Immutable)

## One-Sentence Summary

A string is just an array of characters — so indexing is instant for the same reason arrays are — but because Python strings are *immutable* (they can never be changed in place), the innocent habit of building one with `+` in a loop can quietly become an O(n²) trap that only a fragile CPython optimization sometimes saves you from.

## Why Should You Care?

Text is everywhere: names, URLs, JSON, log files, DNA sequences, this very sentence. Strings are almost certainly the data type you'll touch most in your career. And they hide one of the most common performance bugs that beginners write — and never notice until it's slow in production.

By the end of this article you'll understand:

- Why `name[0]` is instant, exactly like `my_list[0]` (you already know why — it's the same reason).
- Why you *can't* do `name[0] = "J"` in Python, and why that restriction is a feature, not a bug.
- Why building a string piece-by-piece in a loop can quietly cost O(n²), and how a one-line change (`"".join(...)`) makes it robustly O(n) — plus a surprising detail about how CPython sometimes rescues the naive version, and why you still shouldn't rely on that.

This is also where the last three articles pay off at once: contiguous memory (Article 01), the amortized-O(1) append of dynamic arrays (Article 02), and Big-O (Article 03) all show up together. Strings are where the foundations click into place.

## Real-World Analogy

Picture a word spelled out on a row of **stone tablets**, one carved letter per tablet, lined up left to right: `[H][E][L][L][O]`. The tablets are a fixed distance apart and numbered, so if someone says "give me tablet 3," you walk straight to it — no reading the others.

Now here's the twist that defines a Python string: **the letters are carved in stone.** You cannot scratch out the `H` on tablet 0 and carve a `J` instead. If you want the word `JELLO`, you must lay out a *brand-new* row of five tablets and carve all five fresh, leaving the original `HELLO` untouched.

That's immutability. Reading any tablet is instant (you know where it is). But *changing* the word means rebuilding the whole row — and if you rebuild it once per letter while spelling out a long message, you're re-carving more and more stone every step. Hold onto that image; it's the entire performance story.

## Problem Statement

We need to represent **text** — sequences of characters — and do two things with it:

1. **Read** it fast: get the character at a position, scan through it, check its length.
2. **Build and transform** it: join pieces together, insert, replace, reverse.

So the questions are:

> How is text stored so that reading a character is as fast as array access? And what actually happens — in memory, and in Big-O terms — when we "modify" a string?

The first answer will feel familiar. The second one contains a trap that has bitten nearly every programmer at least once.

## First-Principles Explanation

Let's build a string up from nothing, reusing what we already know.

**Step 1 — Characters are numbers.** A computer only stores numbers (Article 01: boxes hold numbers). So each character must map to a number, agreed on by a **character encoding**. In the classic **ASCII** scheme, `'A'` is 65, `'B'` is 66, `'a'` is 97, `' '` is 32. So the character `'H'` is really the number 72 sitting in a box. (Python exposes this: `ord('H')` → 72, and `chr(72)` → `'H'`.)

**Step 2 — A string is an array of those numbers.** Line the character-numbers up in **contiguous, equal-sized boxes** (Article 01), and you have a string. Because the boxes are equal-sized and contiguous, the address of character `i` is the same formula as always:

```
address_of(char i) = base_address + i × char_size
```

That's why `text[i]` is **O(1)** — it's the *identical* mechanism as `my_list[i]`. A string *is* an array; it just happens to hold characters. Everything you learned about array indexing transfers for free.

> **A note on Unicode.** ASCII only covers 128 characters — no `é`, `你`, or `🙂`. Modern text uses **Unicode**, which assigns a number ("code point") to every character in every language. Python 3 strings are sequences of Unicode code points. To keep `text[i]` at O(1), Python stores characters in **fixed-width** boxes (1, 2, or 4 bytes each, whichever fits the widest character in that string) — because the O(1) address formula *requires* equal-sized boxes, exactly the lesson from Article 01. (Encodings like UTF-8 use *variable*-width bytes to save space, which is why indexing raw UTF-8 bytes is *not* O(1) — a trade-off worth its own article later.)

**Step 3 — Strings are immutable.** Here's the pivotal design choice. In Python, once a string exists, its contents can **never change**. `text[0] = "J"` raises `TypeError`. Any operation that "changes" a string — uppercasing, replacing, concatenating — actually **builds a new string** and leaves the original alone.

Why forbid changes? Three first-principles reasons:

- **Hashability.** Strings are the most common dictionary keys and set members. A dict finds a key by its *hash* (a number computed from the contents — the subject of a later article). If a string's contents could change, its hash would change, and it would get "lost" in the dict — silent, catastrophic bugs. Immutability guarantees a string's hash is stable forever, so it's safe as a key.
- **Safe sharing.** If strings can't change, two variables (or a thousand) can point at the *same* string in memory with zero risk — nobody can mutate it out from under the others. Python leans on this heavily (it even reuses small strings automatically). Mutable strings would make sharing dangerous.
- **Predictability.** When you pass a string to a function, you *know* it can't come back modified. No defensive copying, no surprises.

The cost of these guarantees is the whole point of this article: **every "modification" is really a full copy — O(n) in the length of the string.** That's fine once. It's a disaster in a loop.

## Mental Model

Hold two facts together:

1. **A string is a read-only array of characters** at a known base address, with a known length. Reading is array-fast: index in O(1), length in O(1), scan in O(n).
2. **There is no "edit."** Every transforming operation returns a *new* string and never touches the old one. To go from `"HELLO"` to `"JELLO"`, Python allocates a fresh 5-character array and copies — it does not poke the first box.

So the one question to ask about any string-building code is the same one from Article 02: *"How many times am I copying the whole thing?"* Read that answer and you've read the Big-O.

## Visual Diagram (ASCII)

The string `"HELLO"` in memory — characters stored as their code numbers, in equal-sized numbered boxes:

```
index:      0     1     2     3     4
          +-----+-----+-----+-----+-----+
"HELLO" → |  H  |  E  |  L  |  L  |  O  |     length = 5
          | 72  | 69  | 76  | 76  | 79  |     <- ASCII code in each box
          +-----+-----+-----+-----+-----+
addr:      100   101   102   103   104        text[3] = base(100) + 3 = addr 103  → O(1)
```

"Changing" `HELLO` into `JELLO` does **not** edit box 0. It builds a whole new row:

```
original (untouched):   [H][E][L][L][O]   still at addr 100, still "HELLO"
new string:             [J][E][L][L][O]   fresh boxes at addr 200 — 5 copies made
```

Every transform = a new row of boxes. Remember that, and the trap below is obvious.

## The Naive Solution: build a string with `+` in a loop

Say we want to build one big string from many pieces — join a list of words, or assemble a report line by line. The instinctive way:

```python
def join_naive(pieces):
    result = ""
    for piece in pieces:
        result = result + piece   # looks innocent...
    return result
```

It reads naturally: start empty, keep tacking the next piece onto the end. And for a handful of pieces it's perfectly fine.

## Problems With the Naive Solution

The problem is invisible until you remember **strings are immutable**. `result = result + piece` cannot append in place — there's no such thing. It must **build a brand-new string** containing everything in `result` *plus* `piece`, copying every character over. The old `result` is thrown away.

So watch the copying grow as `result` gets longer. Suppose each piece is one character and there are `n` of them:

```
step 1:  result "" + "a"      -> copy 0 chars, new length 1
step 2:  result "a" + "b"     -> copy 1 char,  new length 2
step 3:  result "ab" + "c"    -> copy 2 chars, new length 3
...
step n:  result (n-1 chars) + x -> copy n-1 chars, new length n
```

Total characters copied: `0 + 1 + 2 + … + (n−1) ≈ n²/2`. That's **O(n²)** — the *exact same quadratic signature* as the grow-by-one array in Article 02 and the nested loop in Article 03. By this model, building a 100,000-character string does ~5 *billion* character-copies.

The flaw is structural: immutability means `+` must copy the accumulated result, and the result keeps growing, so the per-step cost grows with it. Constant work would be O(n) total; growing work is O(n²).

!!! note "Under the hood: CPython cheats — but don't count on it"

    Here's a genuine surprise: if you *time* the loop above on CPython, it often runs in **O(n)**, not O(n²). CPython has a special optimization — when the left side of `result = result + piece` has exactly **one reference** (just the variable `result`), the interpreter quietly resizes that string *in place* rather than copying it, much like a dynamic array (Article 02).

    But this is a **fragile, implementation-specific detail**, not a language guarantee. It vanishes the instant:

    - the string gains another reference (e.g. you stash each intermediate in a list), or
    - you run on a different Python (PyPy, Jython, older CPython), or
    - the code isn't the exact "reassign the same variable" shape the optimizer recognizes.

    Measured on CPython 3.12: the plain loop grows ~2× per doubling of `n` (linear — optimization active); hold one extra reference to each intermediate and it jumps to ~4× per doubling (**quadratic** — the true copy cost, exposed). `"".join(...)`, by contrast, is **robustly O(n) everywhere**. So the rule stands — build with a list and `join` — and now you know it's the *safe* choice, not merely the fast one.

## The Better Solution: collect in a list, then `join` once

The fix reuses Article 02 directly. A Python **list is mutable** and its `append` is **amortized O(1)** — no full copy each time. So: append all the pieces into a list (cheap), then combine them into a string in **one** final pass with `str.join`, which walks the pieces once and copies each character exactly once — **O(n)** total.

```python
def join_better(pieces):
    parts = []
    for piece in pieces:
        parts.append(piece)   # amortized O(1), no growing copy (Article 02)
    return "".join(parts)     # one O(n) pass builds the final string
```

`"".join(parts)` means "concatenate every item in `parts`, with the empty string `""` between them." It computes the total length up front, allocates the final array **once**, and copies each character a single time. One allocation, `n` copies — O(n) instead of O(n²).

(In real code you'd often skip the loop entirely — `"".join(pieces)` — but the loop version shows the pattern for when you're *generating* pieces, not just joining a ready-made list.)

## Python Implementation

The two builders side by side, plus the everyday operations and their costs:

```python
def join_naive(pieces):
    """O(n²): each += copies the whole growing result (strings are immutable)."""
    result = ""
    for piece in pieces:
        result = result + piece
    return result

def join_better(pieces):
    """O(n): append to a mutable list (amortized O(1)), then one join pass."""
    parts = []
    for piece in pieces:
        parts.append(piece)
    return "".join(parts)


# --- reading is array-fast (immutability doesn't slow reads) ---
text = "HELLO"
text[0]          # 'H'      -> O(1) indexing, same as arrays
len(text)        # 5        -> O(1), length is stored
text[1:4]        # 'ELL'    -> O(k) slice: copies k chars into a NEW string
"L" in text      # True     -> O(n·m) search in the worst case

# --- "modifying" always returns a NEW string; the original is untouched ---
text.lower()     # 'hello'  -> new string, O(n)
text.replace("L", "R")   # 'HERRO' -> new string, O(n)
text[::-1]       # 'OLLEH'  -> reversed copy, O(n)
```

**Line-by-line:**

- `join_naive` — the trap. `result + piece` allocates a new string each iteration and copies the ever-growing `result`. Quadratic.
- `join_better` — `parts.append` leans on the dynamic array's amortized O(1) (Article 02), then `"".join` does a single O(n) build. Linear.
- `text[0]`, `len(text)` — O(1), because a string is an array with a known base and stored length.
- `text[1:4]` — a slice builds a *new* string of the sliced characters: O(k) where k is the slice length (immutability again — it can't hand you a "view" you might mutate).
- `text.lower()`, `.replace()`, `text[::-1]` — all return **new** strings and leave `text` exactly as it was. Each is O(n).

## Dry Run

Let's build `"abcd"` from `["a","b","c","d"]` both ways and count character-copies.

**Naive (`result = result + piece`):**

```
result=""    + "a"  -> copy 0 -> "a"     (0 copies)
result="a"   + "b"  -> copy 1 -> "ab"    (1 copy)
result="ab"  + "c"  -> copy 2 -> "abc"   (2 copies)
result="abc" + "d"  -> copy 3 -> "abcd"  (3 copies)
                             total = 0+1+2+3 = 6 character-copies
```

**Better (`append` then `join`):**

```
parts=[]                          -> ["a"]           (append, no char copy of prior data)
                                  -> ["a","b"]
                                  -> ["a","b","c"]
                                  -> ["a","b","c","d"]
"".join(parts): allocate length-4 string, copy each char once
                             total = 4 character-copies (one pass)
```

Six copies vs. four looks trivial at n=4. But the naive sum is `n²/2` and the join is `n`: at n=1,000 it's ~500,000 vs. 1,000 — a 500× gap that widens without bound. (That `n²/2` is the copy cost the immutability *model* predicts — and exactly what you pay whenever CPython's in-place optimization doesn't apply, per the note above. `join`'s O(n) needs no such asterisk.) Same lesson as Article 02, now in string form.

## Time Complexity

| Operation | Big-O | Why |
|-----------|-------|-----|
| `text[i]` (index) | O(1) | array address formula; equal-sized boxes |
| `len(text)` | O(1) | length is stored, not counted |
| Scan / iterate all chars | O(n) | visit each of n characters once |
| `text[a:b]` (slice) | O(k) | copies k = b−a characters into a new string |
| `a + b` (concatenate) | O(n+m) | new string; copy both operands |
| Build with `+` in a loop | **O(n²)** by the model † | each concat copies the growing result |
| `"".join(list_of_pieces)` | **O(n)**, robustly | one pass, one allocation, each char copied once |
| `sub in text` (search) | O(n·m) | worst case: try match at each of n spots, m long |
| `.lower()`, `.replace()`, `[::-1]` | O(n) | build a new n-length string |

† By the immutability model, repeated `+` is O(n²); CPython optimizes the single-reference case to O(n), but that's fragile and non-portable (see the note under *Problems With the Naive Solution*). `join` is O(n) with no caveats.

The two rows to burn in: **repeated `+` in a loop risks O(n²); `join` is dependably O(n).** That single swap is one of the highest-value habits in everyday Python — and the safe one regardless of interpreter.

## Space Complexity

- A string of `n` characters uses **O(n)** space — one box per character (times the byte-width Python chose for that string).
- **Immutability means transforms cost extra space, not just time.** `text.lower()` doesn't reuse `text`'s memory; it allocates a *second* n-character string. Chaining `s.strip().lower().replace(...)` can briefly create several full copies.
- The naive `+`-loop is a space offender too: each step allocates a new string and discards the previous one, generating O(n) worth of short-lived garbage for the collector to clean up — on top of the O(n²) time.
- `join` is space-efficient: it measures the total length first and allocates the final buffer **once**, no discarded intermediates.

## Edge Cases

- **Empty string `""`.** Length 0, no boxes of content. `""[0]` raises `IndexError` (there is no box 0). `"".join([...])` is the normal, correct way to concatenate.
- **Single character.** In Python there's no separate "char" type — `text[0]` returns a length-1 *string*. `'H'` and `"H"` are the same thing.
- **Immutability surprises.** `text[0] = "J"` raises `TypeError`. `text.replace("H","J")` returns a new string — and if you forget to *assign* it (`text.replace(...)` on its own line), your change vanishes, because the original is untouched. A classic beginner bug.
- **Non-ASCII / multi-"character" symbols.** Python indexes by code point, but some things you *see* as one character (emoji with modifiers, accented letters composed of two code points) may span more than one index. Usually fine to ignore early on, but it's why "reverse a string" can misbehave on exotic Unicode.
- **`+` for a *few* strings is fine.** `first + " " + last` is O(total length), perfectly reasonable. The trap is specifically `+` *inside a loop*, where the copying compounds.

## Common Mistakes

- **Building strings with `+=` in a loop.** The headline mistake — O(n²) by the model, and only *sometimes* rescued by a fragile CPython optimization (which disappears the moment another reference to the string exists, or you switch interpreters). Reach for a list + `join` whenever you're assembling a string piece by piece — it's dependably O(n).
- **Forgetting the result is new.** `s.strip()` / `s.replace(...)` / `s.upper()` return new strings; writing `s.upper()` without `s = s.upper()` does nothing. Immutability means methods never edit in place.
- **Thinking indexing is slow because strings are "special."** It isn't — `text[i]` is O(1), identical to array indexing. Reading is fast; only *building/transforming* costs O(n).
- **Assuming slicing is free.** `text[1:]` in a loop (e.g., to "consume" a string) copies the remaining characters each time — another accidental O(n²). Use an index instead of re-slicing.
- **Confusing bytes and characters.** `str` is text (code points); `bytes` is raw bytes. Mixing them, or assuming one character equals one byte, breaks the moment non-ASCII text appears. Encode/decode explicitly.
- **Using `+` to combine a known handful.** Over-correcting is a mistake too — for two or three fixed pieces, `a + b + c` is clearer than `"".join([a, b, c])`. The list-join pattern earns its keep in *loops*.

## Interview Perspective

Strings are among the most common interview topics, and the concepts here are exactly what's being probed.

- **"Reverse a string" / "check a palindrome."** Trivial with slicing (`s[::-1]`) or two pointers, but the follow-up is the real test: *"What's the complexity?"* — O(n) time, and O(n) space because immutability forces a new string (you can't reverse in place like a list).
- **"Build/format this output string."** If you write `result += ...` in a loop, a sharp interviewer will ask about efficiency — the expected answer is "collect in a list and `join`, to avoid O(n²)." Saying that unprompted signals real understanding; noting that *CPython has a fragile in-place optimization for the simple case, but `join` is the portable guarantee* signals genuine depth.
- **"Why are Python strings immutable?"** — hashability (safe as dict/set keys), safe sharing, and predictability. A strong, first-principles answer stands out.
- **The "string builder" pattern is universal.** Java has `StringBuilder`, C# has `StringBuilder`, Python uses list + `join`, Go uses `strings.Builder`. They all exist for the *same reason*: to avoid the immutable-concatenation O(n²) trap. Naming that shows you see the pattern beneath the language.
- **Two-pointer and sliding-window** string problems (palindrome, anagram, longest-substring) lean on O(1) indexing and O(n) scanning — the properties this article establishes.

## Practice Questions

1. **On paper**, count the total character-copies to build a 6-character string with the naive `+` loop, then with list-`append` + `join`. What's the ratio, and how does it grow at n = 1,000?
2. Write `reverse(s)` two ways: with slicing, and with a manual loop that appends to a list then joins. State the time and space complexity of each. Why can't you reverse a Python string "in place" the way you could a list?
3. `ord` and `chr` convert between a character and its code point. Write `shift(s, k)` that shifts every letter forward by `k` positions in ASCII (a Caesar cipher). What's its complexity? Which string-building pattern will you use, and why?
4. Explain, using immutability, why `text[0] = "J"` fails but `my_list[0] = "J"` succeeds. What property does a list have that a string gives up — and what does the string get in return?
5. You have code that does `s = s[1:]` inside a `while` loop to process a string front-to-back. What's its hidden complexity, and how do you make it O(n)?
6. **Conceptual:** Why does making strings immutable make them *safe* to use as dictionary keys, while lists are *not* allowed as keys? (Hint: think about what a dict computes from a key, and what would happen if the key changed.)
7. **Stretch:** Use `timeit` to measure the naive `+` loop vs `join_better` on 10,000 / 100,000 / 500,000 one-character pieces. You may be surprised — on CPython the naive loop often looks *linear*, because of the in-place optimization from the note above. Now change the naive loop so it also appends each intermediate `result` to a list (giving the string a second reference), and re-measure. Watch the curve bend into a parabola. Explain what changed — and why `join` never has this problem on any interpreter.

## Key Takeaways

- **A string is an array of characters.** Indexing (`text[i]`) and length are O(1) for the exact reason arrays are — equal-sized boxes at a known base (Article 01). Reading text is fast.
- **Characters are numbers** via an encoding (ASCII/Unicode); Python stores them in fixed-width boxes so O(1) indexing holds.
- **Python strings are immutable** — they can never change in place. This buys hashability (safe dict/set keys), safe sharing, and predictability.
- **The price of immutability: every "modification" is a full O(n) copy.** `.lower()`, `.replace()`, slicing, and `+` all return *new* strings and leave the original untouched.
- **Building a string by repeated `+` in a loop is O(n²) by the immutability model** — the same quadratic signature as Articles 02 and 03. CPython optimizes the simplest single-reference case down to O(n), but that's a fragile, non-portable detail. **Collect pieces in a list and `"".join()` once for dependable O(n).** This is the single most valuable string habit.
- **Every language has this pattern** (`StringBuilder`, `strings.Builder`, list+`join`) for the same reason: dodge the immutable-concatenation trap.

---

*Next up: **References, Objects & Mutability in Python** — we just leaned hard on "strings are immutable" and "lists are mutable," and on the idea that variables hold *references* to objects. Time to make that precise: what a variable really is, why `b = a` doesn't copy, and why mutating a list through one name changes it for every name pointing at it. It's the last foundation before we start building real data structures.*
