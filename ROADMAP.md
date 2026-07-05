# Roadmap

This is the master learning sequence for the DSA book. It is a **living document**:
the order is deliberate (each topic depends only on ideas already introduced), but
topics may be added, split, or resequenced as the book grows.

Status legend: ✅ published · ✍️ in progress · ⬜ planned

---

## Part 0 — Foundations

The mental machinery. Before any data structure or algorithm, the reader must
understand how the machine stores things, how we measure "fast," and how Python
actually represents values. Everything later is a consequence of this part.

| #  | Article | Status | One-line purpose |
|----|---------|--------|------------------|
| 01 | How Computer Memory Works (and Why It Makes Arrays Fast) | ✅ | Memory is numbered boxes; `base + i × size` is why indexing is O(1). |
| 02 | Dynamic Arrays (Why `append` Is Fast Anyway) | ✅ | Fixed memory runs out — doubling + amortized O(1) append. |
| 03 | Big-O Notation (How We Measure "Fast") | ✅ | Growth, not stopwatch: O(1)/O(n)/O(log n)/O(n²) from first principles. |
| 04 | Strings (Arrays of Characters, and Why They're Immutable) | ✅ | Strings as arrays; immutability and the cost of `+` in a loop. |
| 05 | References, Objects & Mutability in Python | ⬜ | Names point to objects; why `list` aliasing surprises beginners. |

**Why this order:** Article 01 establishes contiguous memory → 02 shows what happens
when that fixed run must grow (motivating amortized analysis) → 03 formalizes the
"fast/slow" language we've been using informally → 04 applies arrays + complexity to
strings → 05 closes the foundations by explaining how Python variables really work,
which every later structure (linked lists, trees, graphs) depends on.

---

## Part 1 — Linear Structures *(planned)*

Linked lists, stacks, queues, deques. First structures built *on top of* the
foundations above; the array vs. linked-list trade-off is the through-line.

## Part 2 — Searching & Sorting *(planned)*

Binary search, the classic sorts, and why sorting unlocks faster searching.

## Part 3 — Hashing *(planned)*

Hash tables: turning a *value* into an *address* so search becomes O(1) — the payoff
promised back in Article 01.

*(Later parts — trees, heaps, graphs, dynamic programming — will be sketched once
Parts 0–1 are solid.)*
