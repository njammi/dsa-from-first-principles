# FAQ — Reader Questions

Real questions readers have asked about the articles, answered in full. Each entry
links back to the exact article and section that prompted it, so you can read the
answer in context.

Entries are grouped by the article they refer to.

## Article 01 — How Computer Memory Works

### Why is `my_list[5]` instant, but `5 in my_list` slow? { .faq-q }

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

### Why does it matter that every "house" (element) is the same size? { .faq-q }

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

### Where can I go deeper on how the hardware "activates exactly that box" from an address? { .faq-q }

**Source:** [Article 01 · How Computer Memory Works → *First-Principles Explanation*](part-0-foundations/01-how-computer-memory-works.md#first-principles-explanation)
— the article states: *"The memory hardware is wired so that if you hand it an address (a number),
it can activate exactly that box's circuitry directly."* and deliberately says you don't need to
understand the electronics. If you *want* to, here's the real mechanism and where to learn it.

The concept is **address decoding**. Inside a memory chip, an *address decoder* converts the
binary address into exactly one "on" signal. A decoder with a `k`-bit address has `2ᵏ` outputs
and turns on **only one** of them (a "one-hot" output) — that line is the **word line** that
activates precisely one row of memory cells. For example, feeding address `0101` (5) into a
4-to-16 decoder switches on output line 5 and no other. Larger memories split the address into
**row and column** halves and pick the cell at their intersection (a grid/matrix), which is why
access is direct — it's electrical selection, not searching. This is the physical reason
indexing is O(1): the address *is* the selection signal.

**References to go deeper:**

- **[Ben Eater — "RAM" (Build an 8-bit computer)](https://eater.net/8bit/ram)** — hands-on and
  visual: he builds real RAM on breadboards and shows a 4-bit address selecting one of 16 bytes.
  The most beginner-friendly way to *see* address decoding actually happen. (Video series.)
- **[Address decoder — Wikipedia](https://en.wikipedia.org/wiki/Address_decoder)** — the precise
  term and a concise conceptual reference (word lines, one-hot selection, row/column decoding).
- For a textbook treatment, see the memory chapter of *Digital Design and Computer Architecture*
  (Harris & Harris) or *Computer Organization and Design* (Patterson & Hennessy); the free
  **[nand2tetris](https://www.nand2tetris.org/)** course also builds memory from logic gates.

### If the machine works in addresses, how does the CPU know the address of `x` and `total`? { .faq-q }

**Source:** [Article 01 · How Computer Memory Works → *First-Principles Explanation*](part-0-foundations/01-how-computer-memory-works.md#first-principles-explanation)
— the article says: *"Names like `x` and `total` are a convenience for you, the programmer. The
machine works in addresses."*

The key idea: **names never reach the CPU.** `x` and `total` exist only in your source code.
Before the program runs (compiler) or as it's prepared (interpreter), a translator **assigns
each variable a location and rewrites every name into that location.** The machine code the CPU
executes contains *no names at all* — only registers and addresses.

**How the location is chosen.** As the translator reads your code it builds a **symbol table**
mapping each name to a spot, which is usually either a **CPU register** (fastest — no memory
address at all) or a slot on the **stack** named as an **offset** from a pointer (e.g. "`total`
= the bytes at `base_pointer − 4`"). It emits instructions using those, then discards the names.
So `total = total + x` compiles to roughly:

```
mov   eax, [rbp - 4]     ; load total  (address = base pointer − 4)
add   eax, [rbp - 8]     ; add x       (address = base pointer − 8)
mov   [rbp - 4], eax     ; store back into total
```

No `x`, no `total` — just offsets the CPU blindly reads and writes.

**Why offsets, not fixed addresses?** The same function can run many times (even recursively),
each needing a fresh copy of `x`, so the translator records "`x` is 8 bytes below this call's
stack frame" rather than a hardcoded address; at runtime the CPU computes `base_pointer − 8` to
get the real address for *this* call. (Globals, being single and permanent, do get fixed
addresses.) This is the same `base + offset` arithmetic as the array section, one layer down.

**The Python wrinkle** (since this book is Python): Python adds indirection — names live in a
namespace, and CPython compiles *local* variables into a small fixed array of slots, turning
`x` into an integer index (`LOAD_FAST 0`) at compile time; that slot holds a **reference** to
the object on the heap. So even in Python the name is resolved to a numbered location before
execution — the machine still works in addresses/slots, never names.

**In one sentence:** the CPU never looks up `x`; the compiler/interpreter decided its location
in advance (a register or a stack offset), baked it into the instructions, and threw the name
away. To go deeper, see *Computer Systems: A Programmer's Perspective* (Bryant & O'Hallaron) on
stack frames and symbol tables, or Python's `dis` module to see `LOAD_FAST`/`STORE_FAST` yourself.

??? example "Run it yourself — watch Python turn names into numbered slots"

    ```python
    import dis

    def add_them(x):
        total = 0
        total = total + x
        return total

    # The names, in slot order — this is Python's per-function "symbol table":
    print(add_them.__code__.co_varnames)   # ('x', 'total')

    # The bytecode the interpreter actually runs — names become slot numbers:
    dis.dis(add_them)
    ```

    Output (Python 3.12):

    ```
    ('x', 'total')

      4           2 LOAD_CONST               1 (0)
                  4 STORE_FAST               1 (total)

      5           6 LOAD_FAST                1 (total)
                  8 LOAD_FAST                0 (x)
                 10 BINARY_OP                0 (+)
                 14 STORE_FAST               1 (total)

      6          16 LOAD_FAST                1 (total)
                 18 RETURN_VALUE
    ```

    Read the right-hand column: `x` is **slot 0**, `total` is **slot 1** (matching
    `co_varnames`). The interpreter never works with the *names* `x`/`total` — it works with
    the *numbers* `0` and `1`. The `(x)`/`(total)` in parentheses are just a courtesy that `dis`
    prints for humans; the running loop only uses the index. That's the whole point of the line
    from Article 01, made concrete: names are for you, numbered locations are for the machine.
    (Exact opcodes vary a little by Python version, but the name-to-slot resolution is the same.)
