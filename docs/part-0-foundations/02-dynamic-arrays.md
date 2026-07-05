# Dynamic Arrays (Why `append` Is Fast Anyway)

## One-Sentence Summary

A Python list is secretly a fixed-size array that quietly moves into a bigger home whenever it fills up — and by *doubling* its size each time it moves, it makes `append` feel instant even though occasionally it has to copy everything.

## Why Should You Care?

In the last article we learned the machine's dirty secret: an array is a *contiguous* run of equal-sized boxes at a fixed address. That's what makes `my_list[5]` instant.

But "contiguous and fixed" should bother you. If an array is a fixed run of boxes claimed up front, then:

- How can `my_list.append(x)` add a *new* box when there's no guarantee the next box in memory is free?
- Why does Python never make you declare a list's size, when the machine underneath demands one?
- Is `append` actually free? If not, *when* does it cost something — and how much?

Every one of these questions has the same answer, and it's one of the most elegant ideas in all of computer science: **when you run out of room, don't ask for one more box — ask for twice as many.** Understand *why doubling works*, and you'll understand a trick that shows up again in hash tables, string builders, and file buffers for the rest of your career.

## Real-World Analogy

Imagine you run a small tea shop with a **shelf that holds exactly 4 jars**. Business is good, so a 5th tea arrives. The shelf is full. What do you do?

**Bad idea:** Every time a new tea arrives, buy a shelf one slot bigger, move *every* jar over, and throw away the old shelf. Tea #5 → move 4 jars. Tea #6 → move 5 jars. Tea #7 → move 6 jars. You spend your whole life moving jars.

**Clever idea:** When the shelf fills, buy a shelf **twice as big** and move everything once. Now you have 8 slots but only 5 jars — three of them are empty, waiting. Teas #6, #7, #8 just slot straight in, *no moving at all*. Only when #9 arrives do you upgrade again, this time to 16 slots.

You still move jars sometimes. But because each move *doubles* your free space, the moves get rarer and rarer exactly as fast as the shop grows. That's the whole idea of a dynamic array: **occasional expensive moves, spread thin across many cheap insertions.**

## Problem Statement

We want a list you can keep appending to — `append(x)` — without ever declaring its size in advance, and we want appending to be *fast*.

But the machine only gives us **fixed-size arrays**: to allocate memory, you must say up front how many boxes you want, and the boxes must be contiguous. Once claimed, the run cannot simply "grow" — the boxes immediately after it in memory may already belong to something else.

So the problem is a genuine conflict:

> The hardware offers only fixed-size contiguous arrays. We want an unbounded, growable list. How do we build the second thing out of the first?

## First-Principles Explanation

Let's reason it out from nothing, the way it was actually invented.

**Fact 1 — You cannot resize a run of boxes in place.** Memory is one shared row of boxes (Article 01). Your array occupies boxes 100–103. Box 104 might belong to a completely different variable. So you cannot just "extend" into it. To get a bigger array, you must claim a *brand-new, larger* run of boxes somewhere else and **copy** your values into it.

**Fact 2 — Copying costs time proportional to how much you copy.** Moving `n` values into a new array is `n` individual box-writes. That's O(n) work (we'll make "O(n)" precise in the next article; for now read it as "cost grows in step with n").

So every "grow" is expensive. The entire game is: **grow as rarely as possible.** Here are two growth strategies:

**Strategy A — Grow by 1 each time it fills (naive).**
Insert item 1 → capacity 1. Item 2 → full, allocate size 2, copy 1. Item 3 → full, allocate size 3, copy 2. Item 4 → copy 3...
To insert `n` items you copy `1 + 2 + 3 + … + (n-1)` values. That sum is about `n²/2`. Inserting a million items ≈ **half a trillion** copies. Catastrophe.

**Strategy B — Double the capacity each time it fills (dynamic array).**
Capacity goes 1 → 2 → 4 → 8 → 16 → … To reach `n` items, the copies you pay are `1 + 2 + 4 + 8 + … + n`. Here's the beautiful part: **that sum is less than `2n`.** (A geometric series that doubles is dominated by its last term — the final copy of `n` items costs more than *all previous copies combined*.)

So doubling means: across `n` appends, total copying work is under `2n`. Spread over `n` appends, that's **less than 2 copies of work per append, on average** — a constant. The expensive grows are real, but they get exponentially rarer, so their cost dissolves into the average.

This "average cost per operation, spread over a sequence" has a name: **amortized cost.** A dynamic array's `append` is **amortized O(1)**: usually instant, occasionally O(n), but O(1) *on average, guaranteed.*

The key mental shift:

> Don't ask "what does one `append` cost in the worst case?" (O(n) — the grow). Ask "what does a *whole sequence* of appends cost, divided evenly?" (O(1) each). Amortized analysis measures the second thing.

## Mental Model

Hold two numbers in your head for every dynamic array:

- **length** (`len`) — how many boxes you're actually *using*. This is what `len(my_list)` returns.
- **capacity** — how many boxes you actually *own* right now. This is invisible in Python, but it's really there.

Invariant: **length ≤ capacity, always.** The gap between them is your pre-bought empty space — your "room to grow."

- `append` when **length < capacity**: just drop the value in the next empty box and bump length by 1. Cheap. O(1).
- `append` when **length == capacity**: you're full. Allocate a new run of **2 × capacity** boxes, copy all `length` values over, *then* drop the new value in. Expensive. O(n). But it buys you `capacity` more cheap appends before it happens again.

A dynamic array is just a fixed array plus bookkeeping: *"I own this many boxes, I'm using this many, and when I fill up I'll double."*

This directly answers a question from the top of the article: *why does Python never make you declare a list's size, when the machine underneath demands one?* It doesn't escape that demand — **the list object declares the size for you, and silently re-declares it as you grow.** At every instant your list really does own a fixed block of a specific capacity; CPython's list implementation is the one calling the allocator, tracking `length` and `capacity`, and doing the resize-and-copy dance when you overflow. You're relieved of declaring a size not because the requirement went away, but because the *list* took over that job. "Dynamic array" is exactly that: a fixed-size array the runtime keeps re-declaring behind a friendly `.append()`.

## Visual Diagram (ASCII)

Building a list one append at a time. `■` = used box, `·` = owned-but-empty box (spare capacity):

```
append(A)  ->  capacity 1   [A]                     len 1  cap 1   FULL
append(B)  ->  GROW to 2    [A][·]  copy A, add B
                            [A][B]                   len 2  cap 2   FULL
append(C)  ->  GROW to 4    [A][B][·][·]  copy A,B, add C
                            [A][B][C][·]             len 3  cap 4
append(D)  ->  no grow      [A][B][C][D]             len 4  cap 4   FULL
append(E)  ->  GROW to 8    [A][B][C][D][·][·][·][·]  copy A-D, add E
                            [A][B][C][D][E][·][·][·]  len 5  cap 8
append(F)  ->  no grow      [A][B][C][D][E][F][·][·]  len 6  cap 8
append(G)  ->  no grow      [A][B][C][D][E][F][G][·]  len 7  cap 8
append(H)  ->  no grow      [A][B][C][D][E][F][G][H]  len 8  cap 8   FULL
```

Notice the rhythm: the grows (copies) happen at sizes 1, 2, 4, 8… and the *gaps between them keep doubling*. Between the grow at 4 and the grow at 8, you got **four** free instant appends (D, E via the grow, then F, G, H). The bigger the list, the longer the stretches of pure-cheap appends.

## The Naive Solution: grow by one

Let's build the "grow by 1" array first, so we can *feel* why it's slow. We'll simulate a fixed array with a Python list we promise never to `append` to directly — we only ever allocate a fresh one and copy.

```python
class GrowByOneArray:
    def __init__(self):
        self._store = [None] * 0   # a fixed run of 0 boxes
        self._length = 0

    def append(self, value):
        # Full? (We're always exactly full here: capacity == length.)
        if self._length == len(self._store):
            # Allocate ONE more box than before, then copy everything over.
            new_store = [None] * (self._length + 1)
            for i in range(self._length):     # <-- O(n) copy on EVERY append
                new_store[i] = self._store[i]
            self._store = new_store
        self._store[self._length] = value
        self._length += 1
```

## Problems With the Naive Solution

- **Every single append triggers a full copy.** Capacity always equals length, so the array is full *before every insertion*. There is never any spare room.
- **The cost compounds.** Append #1 copies 0, #2 copies 1, #3 copies 2… append #n copies n−1. Total ≈ `n²/2`.
- **It gets worse as it grows** — exactly backwards from what you want. A big list should not be *more* painful to extend than a small one.
- Building a list of 1,000,000 items does ~500,000,000,000 copies. On real hardware, that's the difference between milliseconds and *minutes*.

The flaw is structural: growing by a *constant* amount means the number of grows stays proportional to `n`, and each grow costs up to `n`. Constant + linear cost per grow = quadratic total. We need the grows to become *rare*.

## The Better Solution: double the capacity

One change fixes everything: when full, don't allocate `length + 1` — allocate `2 × capacity` (or start at 1 when empty). Now most appends land in pre-bought empty boxes and cost nothing extra; the copies happen at capacities 1, 2, 4, 8, 16… and their total is bounded by `2n`.

Same idea as the tea shelf: buy *double*, so future arrivals slot in free.

## Python Implementation

```python
class DynamicArray:
    """A growable array built on fixed-size storage — the idea inside a Python list."""

    def __init__(self):
        self._capacity = 1                       # how many boxes we OWN
        self._store = [None] * self._capacity    # the fixed run of boxes
        self._length = 0                         # how many boxes we USE

    def __len__(self):
        return self._length

    def get(self, index):
        if not 0 <= index < self._length:
            raise IndexError("index out of range")
        return self._store[index]                # O(1): direct box access (Article 01)

    def append(self, value):
        if self._length == self._capacity:       # full — time to grow
            self._resize(2 * self._capacity)     # DOUBLE, not +1
        self._store[self._length] = value        # drop value in next empty box
        self._length += 1                        # one more box in use

    def _resize(self, new_capacity):
        new_store = [None] * new_capacity        # claim a bigger contiguous run
        for i in range(self._length):            # copy existing values over
            new_store[i] = self._store[i]
        self._store = new_store                  # old run is abandoned (garbage-collected)
        self._capacity = new_capacity
```

**Line-by-line:**

- `_capacity` / `_store` / `_length` — the whole mental model made concrete: boxes owned, the boxes themselves, boxes used. The invariant `_length <= _capacity` holds after every method.
- `get` — pure Article-01 indexing. O(1), untouched by all the growth machinery. Growing changes *where* the boxes live, never *how* you read them.
- `append` — the heart. **First** check if full; if so, grow; **then** place the value. Placing and bumping the length are both O(1). The `if` is true only on the rare grow steps.
- `_resize` — the expensive part, isolated in one place: allocate a new run, copy the used values, drop the old run. Doubling is the single decision that makes the amortized math work.

## Dry Run

Let's append `A, B, C, D` to a fresh `DynamicArray` and watch `_length`, `_capacity`, and copies:

```
start:            len=0  cap=1   store=[None]

append(A):
  len(0) == cap(1)?  NO  ->  no resize
  store[0]=A         ->  store=[A]        len=1  cap=1

append(B):
  len(1) == cap(1)?  YES ->  resize to 2, copy 1 value (A)
                            store=[A, None]
  store[1]=B         ->  store=[A, B]     len=2  cap=2

append(C):
  len(2) == cap(2)?  YES ->  resize to 4, copy 2 values (A,B)
                            store=[A, B, None, None]
  store[2]=C         ->  store=[A, B, C, None]   len=3  cap=4

append(D):
  len(3) == cap(4)?  NO  ->  no resize   <-- the payoff: free append
  store[3]=D         ->  store=[A, B, C, D]      len=4  cap=4
```

Four appends, only **two** resizes, copying **1 + 2 = 3** values total. Compare the naive version, which would have copied `0+1+2+3 = 6`. The gap looks small at n=4 — but it's the difference between `2n` and `n²/2`, and that gap explodes as n grows.

## Time Complexity

| Operation | Worst case (one call) | Amortized (per call over a sequence) |
|-----------|-----------------------|--------------------------------------|
| `get(i)` / read by index | O(1) | O(1) |
| `append` (doubling) | O(n) — on a resize | **O(1)** |
| `append` (grow-by-1, naive) | O(n) | O(n) — quadratic total |

**Why `append` is amortized O(1):** over `n` appends, the resizes copy `1 + 2 + 4 + … + n < 2n` values total. Divide `2n` of copying across `n` appends → under 2 per append → constant. The occasional O(n) grow is real, but so rare that its cost, averaged in, vanishes to a constant.

**The doubling series, seen plainly:** `1 + 2 + 4 + 8 + 16 = 31`, which is less than `2 × 16`. The last term always exceeds the sum of everything before it — that's *why* the total is bounded by twice the final size, no matter how big n gets.

## Space Complexity

**O(n)** to store `n` items — but with a constant-factor overhead. Because capacity is always a power of two ≥ length, you may own up to nearly **2×** the boxes you're using right at the moment after a doubling. A list of 17 items sits in a run of 32 boxes; 15 are empty.

This is the fundamental trade of the technique: **you spend spare memory to buy fast appends.** The empty boxes aren't waste — they're pre-paid room to grow, the reason the *next* appends are instant. Time and space, traded against each other, as always.

(Real CPython is a little thriftier than pure doubling — it over-allocates by roughly 1/8 using the pattern `0, 4, 8, 16, 25, 35, 46, …` — but the principle is identical: grow by a *factor*, not a constant, to get amortized O(1) appends.)

## Edge Cases

- **Empty array (`_length == 0`).** Starting capacity is 1, so the very first `append` fills it and the *second* triggers the first resize. (Some implementations start capacity at 0 and treat `0 * 2 = 0` as a special case that jumps to 1 — watch for the `2 * 0 == 0` trap, which would loop forever without a "grow to at least 1" rule.)
- **Appending `None`.** `None` is a legitimate value, but we also use `None` to mean "empty box." That's why we track `_length` separately — never rely on scanning for `None` to find the end. Length is the source of truth; the sentinel is not.
- **Reading an unused box.** `get(_length)` or beyond must raise `IndexError`, even though those boxes physically exist and hold `None`. Owning a box ≠ using it.
- **A single append that overflows into a resize** still returns in the same call — the caller never sees the two-phase (resize, then place) work; it just looks like a normal, if occasionally slower, append.

## Common Mistakes

- **Thinking `append` is *always* O(1).** It's *amortized* O(1). Any individual append can be O(n) when it triggers a resize. In latency-sensitive code (audio, games, real-time systems) that occasional spike matters — you'd pre-size the array to avoid mid-loop resizes.
- **Confusing length with capacity.** `len(my_list)` is items used, not memory owned. They're different numbers, and the gap between them is the whole mechanism.
- **Growing by a constant** (`+1`, `+10`, `+100`). *Any* constant increment gives quadratic total cost — bigger constants just delay the pain, they don't cure it. Only growing by a *factor* (×2, ×1.5) yields amortized O(1). This is the single most important takeaway.
- **Assuming `insert(0, x)` is also amortized O(1).** Inserting at the *front* must shift every existing element right by one box — that's O(n) *every time*, resize or not. Cheap appends are a property of adding at the *end*. (This is exactly why the next Part introduces linked lists.)
- **Forgetting the old array is freed.** After a resize, the old run is abandoned and reclaimed by the garbage collector. You don't hold both forever — but you *do* hold both *briefly* during the copy, a real memory spike for huge arrays.

## Interview Perspective

Dynamic arrays are a favorite because the answer reveals whether you understand *amortized analysis* — a concept that separates memorizers from understanders.

- **"Is Python's `list.append` O(1)?"** — Best answer: "Amortized O(1). Individual appends are usually O(1), but when the underlying array fills, it allocates a larger array — CPython over-allocates — and copies everything, which is O(n). Because it grows by a *factor*, those copies total under a constant times n across n appends, so the average is O(1)."
- **"Why doubling specifically, why not grow by a fixed amount?"** — "Growing by a constant makes the number of resizes proportional to n, and each resize costs up to n, giving O(n²) total. Growing by a factor makes resizes exponentially rare — a geometric series bounded by 2n — so the amortized cost is constant. It's the *factor* that matters, not the number 2."
- **Follow-up they love: "What's the space cost?"** — "Up to ~2× the elements right after a resize; you trade spare memory for fast appends."
- **The trap:** claiming appends are worst-case O(1). Interviewers wait to see if you say "amortized." Saying it unprompted is a strong signal.
- **Practical follow-up: "How would you avoid the resize spikes?"** — "Pre-allocate if you know the size" — e.g., building a known-length result — "so you pay one allocation, no mid-loop copies."

## Practice Questions

1. **On paper**, list the capacities a doubling array passes through as you append 20 items starting from capacity 1. At which append numbers does a resize happen? How many total value-copies occurred?
2. Prove to yourself that `1 + 2 + 4 + … + n < 2n` by computing it for n = 8, 16, and 32. What do you notice about the last term versus the sum of all the others?
3. Modify `DynamicArray` to grow by a factor of **1.5** instead of 2 (round up). Is it still amortized O(1)? What does 1.5 buy you compared to 2, and what does it cost? (Hint: think about wasted space vs. resize frequency.)
4. Add a `pop()` method that removes and returns the last element. Should it ever *shrink* the array? If you shrink at "half full," construct a sequence of `append`/`pop` calls that makes it resize on *every* call. How would shrinking at *one-quarter* full fix that?
5. Add `insert(index, value)` and explain, with a dry run, why its worst case is O(n) *regardless* of capacity — and why no growth strategy can fix that.
6. **Conceptual:** CPython uses the growth pattern `0, 4, 8, 16, 25, 35, 46, 58, 72, …` (over-allocating ~12.5%). Why might a real language choose *less* than doubling? What real-world resource is it being careful with?
7. **Stretch:** Instrument `_resize` to count total copies while appending `n` items. Plot copies vs. n for the doubling array and the grow-by-one array. Confirm one is linear and the other quadratic.

## Key Takeaways

- **The machine only gives fixed arrays; a dynamic array is fixed arrays + bookkeeping.** Track two numbers: **length** (used) and **capacity** (owned), with `length ≤ capacity` always.
- **You cannot resize memory in place** — growing means allocating a bigger run and *copying*. Copying is O(n), so the goal is to copy as rarely as possible.
- **Double the capacity when full.** Growing by a *factor* makes resizes exponentially rare; growing by a *constant* is quadratic and a trap.
- **`append` is amortized O(1), not worst-case O(1).** The rare O(n) resize, spread across many cheap appends (total copies < 2n), averages out to constant per append.
- **Amortized thinking:** don't price one operation in isolation — price a whole *sequence* and divide. It's the right lens whenever cheap operations occasionally trigger an expensive one.
- **You trade space for speed:** up to ~2× memory owned buys instant appends. The empty boxes are pre-paid room to grow, not waste.
- **Fast appends are an *end*-of-array property.** Front inserts are still O(n) — the crack that linked lists are designed to fill.

---

*Next up: **Big-O Notation** — we've been saying "O(1)," "O(n)," and "amortized" on intuition. Time to make that language precise: what Big-O really measures, why we throw away constants, and how to look at a loop and *know* its cost.*
