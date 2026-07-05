# Complexity Reference

A lookup table for the Big-O of common data structures and algorithms — and, just as
importantly, **why** each cost is what it is.

!!! tip "How to use this page"

    This is **not** a page to memorize. Per the FAQ ([Should a beginner memorize Big-O
    tables?](faq.md)), the goal is to *understand the mechanism so you can re-derive every
    row yourself*. Use this page two ways: as a quick **lookup**, and as a **check** — cover the
    numbers, predict them from the "why", then compare. If a row surprises you, that's the row to
    go study. The **mechanism is the point**; the numbers are its consequence.

Values are **average case** unless a worst case is called out. This is a **living page** — it
grows as later parts of the book introduce new structures. Rows link to the article where the
structure is taught, or note which part is coming.

## Big-O growth, slowest- to fastest-growing

The order to internalize (from Article 03). Ask *"if I double the input, what happens to the
work?"*

| Class | Name | Double the input → | Everyday example |
|-------|------|--------------------|------------------|
| O(1) | constant | no change | array index; dict lookup |
| O(log n) | logarithmic | +1 step | binary search |
| O(n) | linear | doubles | scan a list |
| O(n log n) | linearithmic | a bit more than doubles | efficient sorting |
| O(n²) | quadratic | ×4 | nested loop over the same data |
| O(2ⁿ) | exponential | squares | brute-force all subsets |
| O(n!) | factorial | explodes | brute-force all orderings |

## Data structures

Average-case cost of the core operations. "Insert/Delete" means *at the position natural to that
structure* (see the mechanism notes for what that means and where it differs).

| Structure | Access | Search | Insert | Delete | Space | Taught in |
|-----------|--------|--------|--------|--------|-------|-----------|
| Array (fixed) | O(1) | O(n) | O(n) | O(n) | O(n) | [Article 01](part-0-foundations/01-how-computer-memory-works.md) |
| Dynamic array (Python `list`) | O(1) | O(n) | O(n) † | O(n) | O(n) | [Article 02](part-0-foundations/02-dynamic-arrays.md) |
| String (immutable) | O(1) | O(n·m) | — (build new, O(n)) | — | O(n) | [Article 04](part-0-foundations/04-strings.md) |
| Singly linked list | O(n) | O(n) | O(1) ‡ | O(1) ‡ | O(n) | Part 1 (planned) |
| Stack (LIFO) | O(n) | O(n) | O(1) | O(1) | O(n) | Part 1 (planned) |
| Queue (FIFO) | O(n) | O(n) | O(1) | O(1) | O(n) | Part 1 (planned) |
| Hash table (`dict` / `set`) | — | O(1) * | O(1) * | O(1) * | O(n) | Part 3 (planned) |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | Later (planned) |
| Binary heap (`heapq`) | O(1) peek | O(n) | O(log n) | O(log n) | O(n) | Later (planned) |

† **Append** to a dynamic array is **amortized O(1)** (the whole point of Article 02); the O(n)
is for inserting/deleting at an *arbitrary* position, which shifts elements. Deleting at the *end*
is O(1).
‡ Linked-list insert/delete is O(1) **once you already hold the node** (e.g. at the head). If you
must first *find* the position, that search is O(n).
\* Hash-table operations are O(1) **average**, but **O(n) worst case** (many collisions, or a
resize). No ordering by key.

### Why these numbers (the mechanism)

Read the "why" and every row above becomes derivable, not memorized:

- **Array / dynamic array** — contiguous equal-sized boxes, so `base + i × size` gives O(1)
  indexing (Article 01); a value search has no address to jump to, so it scans → O(n); inserting
  in the middle must shift everything after it → O(n); `append` doubles capacity occasionally →
  amortized O(1) (Article 02).
- **String** — an immutable array of characters: indexing is O(1) like any array, but every
  "edit" builds a whole new string → O(n) (Article 04); substring search tries up to n positions,
  each up to m long → O(n·m).
- **Singly linked list** — nodes joined by references (Article 05), *not* contiguous, so there's
  no address formula: reaching index i means walking i links → O(n). But splicing a node in or out
  when you already hold it is just a couple of pointer swaps → O(1). It trades the array's fast
  indexing for fast insertion. (This is Part 1's central trade-off.)
- **Stack / queue** — restricted linked structures (or dynamic arrays) where you only ever touch
  the end(s): push/pop/enqueue/dequeue are O(1). You give up random access (O(n)) in exchange.
- **Hash table** — computes an *address from the value itself* (hashing), turning search into
  near-direct access → ~O(1) average; collisions and occasional resizes give the O(n) worst case.
  This is the O(1) search promised back in Article 01.
- **Balanced BST** — a sorted tree kept ~log n tall; each comparison discards half the remaining
  nodes → O(log n) per operation, and an in-order walk yields sorted output in O(n). (An
  *unbalanced* BST degrades to a linked list → O(n).)
- **Binary heap** — a complete binary tree stored in an array; the min/max sits at the root
  (O(1) peek), and push/pop restore order by sifting along the height → O(log n).

## Sorting algorithms

Time by case, extra space, and stability (whether equal elements keep their original order).

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble sort | O(n) | O(n²) | O(n²) | O(1) | yes |
| Insertion sort | O(n) | O(n²) | O(n²) | O(1) | yes |
| Selection sort | O(n²) | O(n²) | O(n²) | O(1) | no |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) | yes |
| Quicksort | O(n log n) | O(n log n) | O(n²) | O(log n) | no |
| Heapsort | O(n log n) | O(n log n) | O(n log n) | O(1) | no |
| **Timsort** (Python's `sorted` / `list.sort`) | O(n) | O(n log n) | O(n log n) | O(n) | yes |

Notes: **O(n log n) is the best any comparison sort can do** in the worst case — merge, heap, and
Timsort hit it guaranteed; quicksort averages it but degrades to O(n²) on bad pivots. **Timsort**
is what Python actually uses: a hybrid of merge and insertion sort, stable, and O(n) on
already-sorted data. Insertion sort's O(n) best case is why real sorts switch to it for small
chunks. *(Sorting gets its own treatment in Part 2.)*

## Searching algorithms

| Algorithm | Time | Space | Requires |
|-----------|------|-------|----------|
| Linear search | O(n) | O(1) | nothing — works on unsorted data |
| Binary search | O(log n) | O(1) | data must be **sorted** |

Binary search halves the range each step (Article 03), which is *why* it's O(log n) — but it only
works on sorted data, which is a big part of *why sorting matters*: sort once (O(n log n)), then
search many times in O(log n). *(Both covered in Part 2.)*

---

*Remember the FAQ's point: the win isn't reciting this page — it's being handed **arbitrary code**
and reading its Big-O straight off, because you understand the mechanisms these numbers come from.*
