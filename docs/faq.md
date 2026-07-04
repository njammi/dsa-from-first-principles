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
