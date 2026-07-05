# References, Objects & Mutability in Python

## One-Sentence Summary

In Python a variable is not a box that holds a value — it's a *name tag* stuck onto an object that lives elsewhere in memory, and once you see that, every confusing bug about "why did changing one list change the other?" instantly makes sense.

## Why Should You Care?

This is the article that stops a whole category of baffling bugs — the kind that make you distrust the language itself:

```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)      # [1, 2, 3, 4]  ... wait, I never touched a?!
```

If that output surprises you, you're holding the wrong mental model of what a variable *is* — and you're in excellent company, because almost everyone starts there. By the end of this article you'll understand:

- Why `b = a` does **not** copy the list, and when that bites you.
- Why `x = 5; y = x; x += 1` leaves `y` at 5, but the list example above behaves differently — and why that's actually the *same* rule, not two.
- Why passing a list to a function can modify it, but passing a number can't.
- The single most infamous Python trap — the "mutable default argument" — and why it happens.

This also completes Part 0. We've been quietly relying on "variables hold references" (Article 01) and "strings are immutable, lists are mutable" (Article 04). This article makes all of that precise — the last foundation before we build real data structures.

## Real-World Analogy

Imagine objects — a list, a number, a string — as **boxes sitting in a warehouse**. A variable is not one of those boxes. A variable is a **name tag on a string** that you tie to a box.

- `a = [1, 2, 3]` creates a box (the list) in the warehouse and ties the name tag `a` to it.
- `b = a` does **not** build a second box. It ties *another* name tag, `b`, to the **same** box. Now two tags, one box.
- `b.append(4)` walks up to whatever box `b` is tied to and drops a `4` inside it. Since `a` is tied to that very same box, `a` "sees" the change — of course it does, there's only one box.
- `b = [9, 9]` doesn't touch the old box at all. It **re-ties** the tag `b` to a *different* box. `a`'s tag hasn't moved, so `a` still sees the original.

The entire article is that picture. "Assignment ties a name to an object." "Mutation changes the object a name is tied to." "Rebinding moves the name to a different object." Confusion always comes from mixing up *moving a tag* with *changing a box*.

## Problem Statement

We need a precise answer to a question beginners get wrong constantly:

> When I write `b = a`, and then change something through `b`, what happens to `a`? And why does the answer seem to depend on whether it's a list or a number?

Get this right and aliasing bugs, function-argument surprises, and copy confusion all dissolve. Get it wrong and Python feels arbitrary and haunted.

## First-Principles Explanation

Let's build the correct model from the ground up.

**Fact 1 — Everything is an object, and objects live in memory.** In Python, *every* value — a list, an integer, a string, even a function — is an object sitting at some address in memory (Article 01: memory is addressed boxes). Each object knows its type and its value.

**Fact 2 — A variable is a name bound to a reference.** A variable does not *contain* an object. It holds a **reference** — essentially the object's address (Article 01) — and the name is *bound* to that reference. This is exactly the "array of references" idea from Article 04: names point to objects; the objects live elsewhere.

So `a = [1, 2, 3]` does two things: it creates the list object somewhere in memory, and it binds the name `a` to that object's reference.

**Fact 3 — Assignment copies the reference, never the object.** `b = a` copies the *reference* — the address — into `b`. It does not duplicate the list. Both names now hold the same reference, so both point at the **one** underlying object. You can check this: `id(a)` and `id(b)` (an object's identity, essentially its address) are **equal**, and `a is b` is `True`. There is one object with two names — an **alias**.

**Fact 4 — Now the pivotal split: mutate vs. rebind.**

- **Mutating** changes the *object itself*, in place. `b.append(4)` reaches through `b` to the shared list object and modifies it. Every name bound to that object sees the change, because there's only one object. This is why `a` changed.
- **Rebinding** points a *name* at a different object. `b = [9, 9]` makes a new list and binds `b` to it. The old object is untouched; `a` still points to it. Only `b` moved.

**Fact 5 — Mutability decides which operations are even possible.**

- **Mutable** objects can be changed in place: `list`, `dict`, `set`. So aliasing them is observable — change through one name, see it through another.
- **Immutable** objects cannot be changed in place: `int`, `float`, `str` (Article 04!), `tuple`, `bool`. There is *no operation* that mutates them. So every "change" to an immutable is secretly a **rebind** to a new object.

This is why `x = 5; y = x; x += 1` leaves `y == 5`. Integers are immutable, so `x += 1` can't modify the `5` object — it creates a new `6` object and rebinds `x` to it. `y` still points at the original `5`. It's the **same rule** as the list case; it only *looks* different because immutables force rebinding while mutables permit true mutation. One rule, two appearances.

## Mental Model

Three sentences hold everything:

1. **A name is a tag; an object is a box.** Assignment (`=`) ties a tag to a box. Multiple tags can share one box (aliases).
2. **Mutation changes the box.** Every tag on that box sees it. Only possible for mutable types (`list`, `dict`, `set`).
3. **Rebinding moves a tag to another box.** No other tag is affected. Every "change" to an immutable type (`int`, `str`, `tuple`, …) is really this.

Whenever code surprises you, ask: *"Did I change the box, or move the tag?"* That question resolves essentially every aliasing puzzle in Python.

## Visual Diagram (ASCII)

Start:

```
a ──▶ ┌───────────┐
b ──▶ │ [1, 2, 3] │   one list object, two name tags (a is b → True)
      └───────────┘
```

After `b.append(4)` — **mutate the shared box**:

```
a ──▶ ┌──────────────┐
b ──▶ │ [1, 2, 3, 4] │   same object, changed in place — BOTH names see it
      └──────────────┘
```

After `b = [9, 9]` — **rebind the tag `b`**:

```
a ──▶ ┌──────────────┐
      │ [1, 2, 3, 4] │   a still here, untouched
      └──────────────┘
b ──▶ ┌──────────┐
      │ [9, 9]   │       b now points at a NEW object
      └──────────┘
```

The immutable case, `x = 5; y = x; x += 1`:

```
x ──▶ ┌───┐        x += 1 can't change the 5 (ints are immutable),
y ──▶ │ 5 │   ─▶   so it makes a new 6 and moves ONLY x:
      └───┘
                   x ──▶ ┌───┐      y ──▶ ┌───┐
                         │ 6 │            │ 5 │
                         └───┘            └───┘
```

## The Naive Model: "a variable is a box that holds a value"

Most beginners arrive with this model (often from math, or from languages like C where it's closer to true): a variable is a little box, and assignment *puts a value into the box*. `a = [1,2,3]` means "box `a` contains `[1,2,3]`." `b = a` means "copy `a`'s contents into box `b`."

It's intuitive, and for simple immutable values it even gives the right answers.

## Problems With the Naive Model

The box-holds-a-value model **cannot explain** the behavior you actually observe:

- It predicts `b = a; b.append(4)` leaves `a` alone (separate boxes, separate contents). But `a` *does* change. The model is simply wrong here.
- It can't explain why the *same* `b = a` pattern "copies" for integers (`y` stays 5) but "shares" for lists. Under the box model these should behave identically — but they don't.
- It offers no account of why passing a list into a function can modify the caller's list, while passing an integer can't.

When a model makes wrong predictions, you don't patch it — you replace it. The name-tag/reference model above makes *every one* of these predictions correctly, with a single rule. That's the sign of the right model.

## The Better Model: names → references → objects

Replace "box holds a value" with: **a name is bound to a reference that points to an object.** Assignment copies the *reference*; mutation changes the *object*; rebinding moves the *name*. Mutable objects can be mutated (so aliases are observable); immutable objects can't (so every change is a rebind). One rule, and all the confusing cases fall out correctly — as the code below demonstrates.

## Python Implementation

Python gives you tools to *see* references directly: `id(obj)` returns an object's identity (think "its address"), and `is` tests whether two names point to the **same object** (identity), as opposed to `==`, which tests whether two objects have the same **value** (equality).

```python
# --- aliasing: one object, two names ---
a = [1, 2, 3]
b = a                 # copy the REFERENCE, not the list
print(a is b)         # True  -> same object
print(id(a) == id(b)) # True  -> same identity/address

b.append(4)           # MUTATE the shared object
print(a)              # [1, 2, 3, 4]  -> a sees it, because it's the same object

b = [9, 9]            # REBIND b to a new object
print(a)              # [1, 2, 3, 4]  -> a unaffected; only b moved
print(a is b)         # False

# --- immutables force rebinding ---
x = 5
y = x                 # both names point to the same 5
x += 1                # can't mutate 5; makes a new 6 and rebinds x
print(x, y)           # 6 5  -> y still points at the original 5

# --- is vs == ---
p = [1, 2]
q = [1, 2]
print(p == q)         # True  -> same value
print(p is q)         # False -> different objects

# --- function arguments: Python passes the reference by value ---
def add_item(lst):
    lst.append("!")   # mutates the caller's object (same reference)

def rebind(lst):
    lst = ["new"]     # rebinds the LOCAL name only; caller unaffected

items = ["a"]
add_item(items)
print(items)          # ['a', '!']  -> mutation is visible to the caller
rebind(items)
print(items)          # ['a', '!']  -> rebinding inside the function is not
```

**Line-by-line:**

- `b = a` copies the reference; `a is b` and equal `id`s prove there's one object.
- `b.append(4)` mutates that shared object — `a` changes too.
- `b = [9, 9]` rebinds `b`; `a` keeps pointing at the original.
- `x += 1` on an immutable int rebinds `x` to a fresh object, leaving `y`.
- `p is q` is `False` though `p == q` is `True`: equal value, different objects.
- In `add_item`, the parameter `lst` is bound to the *same* object as `items`, so mutating it is visible to the caller. In `rebind`, `lst = [...]` only re-points the local name — the caller's binding is untouched. This is often called **"pass by object reference"** (or "call by sharing"): the function receives a copy of the *reference*, not a copy of the object, and not the caller's variable itself.

## Dry Run

Trace the aliasing sequence with illustrative addresses (real `id()` values differ per run, but the *relationships* are what matter):

```
a = [1, 2, 3]      a → obj@0x100  [1,2,3]
b = a              b → obj@0x100         (copied the reference; a is b → True)

b.append(4)        obj@0x100 becomes [1,2,3,4]      (mutated in place)
                   a → obj@0x100  [1,2,3,4]   b → obj@0x100  [1,2,3,4]
                   both names, one object → both see [1,2,3,4]

b = [9, 9]         new object created
                   a → obj@0x100  [1,2,3,4]         (unchanged)
                   b → obj@0x200  [9,9]             (b rebound; 0x100 untouched)
```

And the immutable trace:

```
x = 5              x → int@0x008 (5)
y = x              y → int@0x008 (5)         (same object; x is y → True)
x += 1             int 5 can't change → new int 6 created
                   x → int@0x010 (6)   y → int@0x008 (5)
                   only x moved; y still 5
```

The rule never changed: mutation edits the shared object; assignment/rebind moves a name.

## Time Complexity

- **Assignment `b = a`: O(1).** It copies a single reference (one address), regardless of how big the object is. Aliasing a million-element list is as cheap as aliasing an empty one — no data is copied. (This is *why* Python can pass large objects to functions cheaply.)
- **Mutation** cost depends on the operation, not on references: `list.append` is amortized O(1) (Article 02); `list.insert(0, x)` is O(n).
- **Shallow copy** (`list(a)`, `a[:]`, `copy.copy(a)`): **O(n)** — copies the n references, but not the objects they point to.
- **Deep copy** (`copy.deepcopy(a)`): **O(total size)** — recursively copies every nested object. Much more expensive; use only when you truly need independent nested data.

## Space Complexity

- **Aliasing costs no extra space** — a second name is just another reference to the same object. This is a feature: Python shares freely, which is *safe precisely because* immutable objects can't be mutated out from under you (Article 04's payoff).
- **A shallow copy** allocates a new container of n references (O(n) space) but *shares* the nested objects with the original. Mutating a shared nested object shows through both copies — the classic shallow-copy gotcha.
- **A deep copy** duplicates everything, using space proportional to the entire nested structure — full independence, at full cost.

## Edge Cases

- **Small-integer & string caching.** CPython pre-creates and reuses small integers (−5 to 256) and interns many short strings, so `a = 256; b = 256; a is b` is reliably `True`. For larger integers it's **unreliable and context-dependent** — `a = 1000; b = 1000; a is b` can be `True` *or* `False` depending on how and where the literals are compiled (both in one code block often reuse the constant; entered separately in the REPL they usually don't). The lesson is the same either way: this is an implementation optimization, so **never use `is` to compare values.** Use `==` for value, and `is` only for identity (notably `x is None`).
- **`is None` is the correct idiom.** There's exactly one `None` object, so identity is the right, fast test. Prefer `if x is None:` over `if x == None:`.
- **Tuples are immutable, but can hold mutable objects.** `t = ([1, 2], 3)` — you can't rebind `t[0]`, but you *can* do `t[0].append(9)`, mutating the list inside the "immutable" tuple. Immutability is shallow: the tuple's *references* are fixed, not the objects they point to.
- **Rebinding vs. mutating an argument** inside a function is the crux of most "why didn't my change stick?" bugs — `param = ...` never affects the caller; `param.append(...)` does.
- **Empty containers still have identity.** Two separate `[]` are distinct objects (`[] is []` → `False`), even though they're equal.

## Common Mistakes

- **Accidental aliasing.** `b = a` (for a list/dict/set) then mutating `b` changes `a`. To get an independent copy, say so explicitly: `b = a[:]`, `b = list(a)`, or `copy.deepcopy(a)` for nested structures.
- **The mutable default argument trap.** `def f(x, acc=[]):` — the default list is created **once**, when the function is defined, and *reused across every call*, silently accumulating. The fix is the standard `None` sentinel:

```python
# BUG: the same list is shared by every call that uses the default
def append_to(value, target=[]):
    target.append(value)
    return target

print(append_to(1))   # [1]
print(append_to(2))   # [1, 2]   <- surprise! the default kept the old data

# FIX: default to None, create a fresh list inside
def append_to_fixed(value, target=None):
    if target is None:
        target = []
    target.append(value)
    return target

print(append_to_fixed(1))   # [1]
print(append_to_fixed(2))   # [2]   <- fresh each call
```

- **Shallow-copying nested structures.** `b = a[:]` copies the outer list, but the *inner* lists are shared. `b[0].append(9)` still shows up in `a[0]`. Use `copy.deepcopy` when you need the nesting to be independent.
- **Using `is` for value comparison.** `if count is 0:` or `if name is "yes":` may work by accident (caching) and then fail unpredictably. Use `==` for values; reserve `is` for identity (`is None`).
- **Expecting a function to "copy" its argument.** It receives the reference. Mutating the object affects the caller; if you need to protect the caller's data, copy inside the function.

## Interview Perspective

This topic is a favorite because it separates people who *use* Python from people who *understand* it.

- **"Is Python pass-by-value or pass-by-reference?"** — The precise answer: neither of the naive versions — it's **"pass by object reference"** (call by sharing). The function gets a copy of the *reference*. Mutating the object is visible to the caller; rebinding the parameter is not. Explaining it with a mutate-vs-rebind example is a strong signal.
- **"What does `a is b` mean vs `a == b`?"** — `is` = same object (identity); `==` = same value (equality). Mentioning that `is` should be used for `None` and *not* for value comparison (because of int/str caching) shows real depth.
- **The mutable-default-argument question** appears constantly — be ready to explain *why* (the default is evaluated once at definition) and give the `None`-sentinel fix.
- **"How do you copy a list/dict safely?"** — shallow (`[:]`, `list()`, `copy.copy`) vs deep (`copy.deepcopy`), and *why* shallow isn't enough for nested data.
- **Why lists can't be dict keys but tuples (of immutables) can** — keys must be hashable, which requires a stable hash, which requires immutability (Article 04). A perfect place to connect this article to the last one.

## Practice Questions

1. **On paper**, predict the output, then run it: `a = [1, 2]; b = a; c = a[:]; b.append(3); c.append(4); print(a, b, c)`. Explain each result in terms of tags and boxes.
2. Predict `x = [0]; y = x; x = x + [1]; print(x, y)` versus `x = [0]; y = x; x += [1]; print(x, y)`. They differ! Explain why (hint: `x + [1]` builds a new list and rebinds; `x += [1]` mutates in place for lists).
3. Write a function that takes a list and returns a *sorted copy* without modifying the caller's list. Then write one that sorts the caller's list in place. What's the one-line difference, and how does each treat the reference?
4. Demonstrate the mutable-default-argument bug with your own function, then fix it with the `None` sentinel. Explain when the default list is created.
5. Given `t = (1, [2, 3])`, which of these work and which raise, and why: `t[0] = 9`, `t[1] = [9]`, `t[1].append(9)`? What does this reveal about "immutable"?
6. **Conceptual:** Using the reference model, explain why aliasing a huge list is O(1) space and time, but `copy.deepcopy` of it is O(total size). When would each be the right choice?
7. **Stretch:** Use `id()` to *prove* that `x += 1` on an integer creates a new object (the id changes) while `lst.append(1)` on a list does not (the id stays the same). Write the few lines and interpret the ids.

## Key Takeaways

- **A variable is a name bound to a reference to an object — not a box holding a value.** Assignment (`=`) copies the *reference*, never the object.
- **`b = a` creates an alias:** two names, one object (`a is b` → `True`). No data is copied — assignment is O(1) no matter the object's size.
- **Mutate vs. rebind is the whole game.** *Mutating* (`b.append(...)`) changes the shared object — every name sees it. *Rebinding* (`b = ...`) moves one name to a new object — no other name is affected.
- **Mutability decides what's possible.** `list`/`dict`/`set` can be mutated (aliases are observable); `int`/`str`/`tuple` are immutable, so every "change" is really a rebind — which is why `y` stays 5 in the integer example. *One rule, two appearances.*
- **Functions pass object references by value** ("call by sharing"): mutating an argument affects the caller; rebinding the parameter does not.
- **`is` = identity, `==` = value.** Use `is` only for identity (especially `is None`); never for comparing values (int/str caching will betray you).
- **Copy deliberately:** shallow (`[:]`, `list()`) shares nested objects; `copy.deepcopy` makes them independent. And beware the mutable-default-argument trap — use the `None` sentinel.

---

*That completes **Part 0 — Foundations**. You now hold the whole machine in your head: memory as addressed boxes (01), arrays that grow (02), Big-O to measure cost (03), strings as immutable character arrays (04), and names-as-references to mutable-or-immutable objects (05). Everything from here is built out of these five ideas.*

*Next up: **Part 1 — Linear Structures**, starting with the **Linked List** — our first data structure built by hand. It throws away contiguous memory entirely and connects objects with references (exactly what you just learned), trading the array's O(1) indexing for O(1) insertion anywhere. The great array-vs-linked-list trade-off begins.*
