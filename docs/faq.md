# FAQ — Reader Questions

Real questions readers have asked about the articles, answered in full. Each entry
links back to the exact article and section that prompted it, so you can read the
answer in context.

Entries are grouped by the article they refer to.

---

## Article 01 — How Computer Memory Works

### Why is `my_list[5]` instant, but `5 in my_list` slow?

**Source:** [Article 01 · How Computer Memory Works → *Why Should You Care?*](part-0-foundations/01-how-computer-memory-works.md#why-should-you-care)
— the article poses this as a teaser: *"Why is `my_list[5]` instant, but `5 in my_list` slow?"*

The subtlety is that the `5` means two **completely different things** in those two lines.

**`my_list[5]` — the `5` is a *position* (an index).** You already know *where* to look.
From Article 01, memory is a row of equal-sized, numbered boxes at a known base address,
so Python finds the box with one arithmetic formula:

```
address = base_address + 5 × element_size
```

That's a single multiply-and-add — a constant amount of work — then it reads that one box
directly. Same speed whether the list holds 10 items or 10 million. This is **O(1)**. Boxes
0–4 are never touched; Python jumps straight to box 5.

**`5 in my_list` — the `5` is a *value* (a thing to find).** You do **not** know where it is —
that's what you're trying to discover. A value tells you nothing about its address, so there
is no formula to jump to. Python must check the boxes one at a time:

```
box 0 == 5?  no
box 1 == 5?  no
...keep going until found, or until the list ends
```

For `n` elements, the worst case (the value is last, or absent) is `n` comparisons — **O(n)**,
and it gets slower as the list grows.

**In one sentence:** indexing knows the *position* and computes the *address* in one step
(O(1)); searching knows only the *value* and must *scan* to discover the position (O(n)). Same-
looking syntax, opposite costs — the difference is entirely *"do I already know where it is,
or do I have to go find it?"* (This is the "index ≠ address" idea from Article 01, expressed in
the Big-O vocabulary of [Article 03](part-0-foundations/03-big-o-notation.md).) It's also why
**hash tables** feel magical: they make `x in my_set` behave like *indexing* instead of
*searching*, by computing an address *from the value itself* — turning the O(n) scan back into
an ~O(1) jump.

### Why does it matter that every "house" (element) is the same size?

**Source:** [Article 01 · How Computer Memory Works → *Real-World Analogy*](part-0-foundations/01-how-computer-memory-works.md#real-world-analogy)
— from the street-of-houses analogy: *"Every house is the same size. No mansions, no cottages
— identical boxes."*

Because equal size is the one assumption that makes the address formula work:

```
address = base_address + index × element_size
```

The `index × element_size` is a **multiplication**, and multiplication is just repeated
addition of an *identical* quantity. It only means "skip over `index` houses" if every house
is the same width. Same size → you can multiply → one arithmetic step → **O(1)** access.

If houses were different sizes (house 0 takes 3 boxes, house 1 takes 1, house 2 takes 8…),
there's no formula. To find house 5 you'd have to add up the actual sizes of houses 0–4 to
know where house 5 even *starts* — and to know those sizes you'd have to go *look at each one*.
That's a scan of everything before it: **O(n)**, not a single jump. So:

> Same size → multiply → one step → O(1). Different sizes → sum the ones before → walk them → O(n).

"Every house is the same size" isn't a throwaway detail — it *is* the exact property that buys
constant-time random access. This is also **why Python lists store references** (pointers), not
the values themselves: a string might need 50 bytes and an integer 28 — different sizes, which
would break the formula — but a *reference* to each is always the same size, so Python keeps an
array of equal-sized references and lets the values live elsewhere. (Article 01 hints at this in
its stretch practice question about why lists store references.)
