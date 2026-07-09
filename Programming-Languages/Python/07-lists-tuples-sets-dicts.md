---
title: "Lists, Tuples, Sets, and Dicts"
created: 2026-07-08
updated: 2026-07-08
tags: [programming-languages, python, collections, list, dict, set, tuple]
aliases: ["Python collections", "Python data structures built-in", "list tuple set dict"]
---

# Lists, Tuples, Sets, and Dicts

[toc]

> **TL;DR:** Python’s everyday data structures are **list**, **tuple**, **set**, and **dict**. All of them store **references** to objects. Choose by job: ordered growable sequence, fixed record, unique membership, or key→value lookup. Cost and gotchas follow from that model—not from “variables holding values.”

---

## 1. The four jobs (start here)

You almost always want one of these four. Note [04](./04-basic-syntax-and-data-types.md) introduced them; this note is the working manual.

![Four containers](./assets/07-lists-tuples-sets-dicts/four-containers.svg)

| Need | Reach for | Why |
| :--- | :--- | :--- |
| Sequence I will change | `list` | Index + append/pop in place |
| Fixed record / “don’t mutate the structure” | `tuple` | Immutable slots; can be dict keys if hashable |
| “Have I seen this?” / unique items | `set` | Hash membership, avg O(1) |
| Lookup by name/id | `dict` | Hash map, avg O(1) get/set |

> [!TIP]
> Default habit for beginners: everything is a list. Prefer a **dict** for named fields and a **set** for membership checks once the data grows.

---

## 2. List — ordered, mutable sequence

A **list** is a dynamic array of **references**. Indexing is O(1); membership (`x in xs`) walks the whole list (O(n)).

![List of references](./assets/07-lists-tuples-sets-dicts/list-refs.svg)

```python
nums = [10, 20, 30]
nums.append(40)       # [10, 20, 30, 40]
nums.insert(0, 5)     # shifts everything right — O(n)
nums.pop()            # remove last — O(1) amortized
nums[1:3]             # slice → new list [20, 30]
```

### Everyday operations

| Op | Example | Cost (typical) |
| :--- | :--- | :--- |
| Index | `xs[i]` | O(1) |
| Slice | `xs[a:b]` | O(b−a) new list |
| Append | `xs.append(x)` | O(1) amortized |
| Pop last | `xs.pop()` | O(1) |
| Pop/insert front | `xs.pop(0)`, `insert(0, x)` | O(n) |
| Membership | `x in xs` | O(n) |
| Sort in place | `xs.sort()` | O(n log n) |
| Sorted copy | `sorted(xs)` | O(n log n) + new list |

```python
# stack (end of list)
stack = []
stack.append(1)
stack.pop()

# reverse iterate without copying
for x in reversed(nums):
    ...

# sort key
people.sort(key=lambda p: p["age"])
```

### List comprehensions

Build a new list in one expression. Prefer them when the body is a short map/filter.

```python
squares = [x * x for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]
```

> [!NOTE]
> A comprehension **allocates the full result**. For huge streams, use a generator expression `(...)` and consume it (see [05](./05-conditionals-and-loops.md)).

---

## 3. Tuple — ordered, immutable sequence

A **tuple** has a fixed number of slots. You cannot reassign `t[i]` or change length. Elements can still be **mutable objects** (the slots hold references).

```python
point = (3, 4)
# point[0] = 1   # TypeError

t = ([1], 2)
t[0].append(9)   # OK — same list object inside
# t → ([1, 9], 2)
```

### Why use tuples?

| Use | Example |
| :--- | :--- |
| Multiple return values | `return lat, lon` |
| Dict / set keys | `seen.add((r, c))` |
| Unpack | `x, y = point` |
| Heterogeneous record | `("ada", 36, True)` |

```python
def minmax(xs):
    return min(xs), max(xs)

lo, hi = minmax([3, 1, 4])
```

**Hashable:** a tuple is hashable only if **all** elements are hashable. `([1],)` is not a valid dict key.

Single-element tuple needs a trailing comma: `(1,)` not `(1)`.

---

## 4. Set — unique hashable items

A **set** is an unordered collection of unique **hashable** elements. Membership and add/remove are average O(1).

```python
tags = {"python", "go", "python"}   # {"python", "go"}
tags.add("rust")
"go" in tags                        # True

# empty set — not {}
empty = set()
```

### Set algebra (the good part)

```python
a = {1, 2, 3}
b = {2, 3, 4}

a | b    # union {1, 2, 3, 4}
a & b    # intersection {2, 3}
a - b    # difference {1}
a ^ b    # symmetric difference {1, 4}
```

| Goal | Pattern |
| :--- | :--- |
| Dedupe preserve nothing | `set(items)` |
| Dedupe keep order (3.7+) | `list(dict.fromkeys(items))` |
| Fast membership | build a `set`, then `x in s` |
| Immutable set | `frozenset(...)` — hashable, usable as dict key |

> [!IMPORTANT]
> Lists and other unhashable types cannot go in a set. Convert to tuple (if elements are hashable) or use another structure.

---

## 5. Dict — key → value map

A **dict** maps **hashable keys** to values (any objects). CPython dicts preserve **insertion order** (language guarantee since 3.7).

![Dict hash lookup](./assets/07-lists-tuples-sets-dicts/dict-hash.svg)

```python
user = {"id": 1, "name": "ada"}
user["name"] = "Ada"
user["email"] = "ada@example.com"   # insert

user.get("phone")                   # None if missing
user.get("phone", "n/a")            # default

for key, value in user.items():
    print(key, value)
```

### Views and safe access

| API | Meaning |
| :--- | :--- |
| `d.keys()` | live view of keys |
| `d.values()` | live view of values |
| `d.items()` | live view of (k, v) pairs |
| `d.get(k, default)` | no `KeyError` |
| `d.setdefault(k, v)` | get or insert default |
| `d \| other` (3.9+) | merge, right wins |
| `{**a, **b}` | merge into new dict |

```python
# merge (3.9+)
merged = defaults | overrides

# count with dict
counts: dict[str, int] = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

# or collections.Counter for that job
```

**Hashable keys:** `str`, `int`, `float` (careful with NaN), `bool`, `tuple` of hashables, `frozenset`. Not `list`, `dict`, `set`.

---

## 6. Choosing and converting

```python
list("ab")              # ["a", "b"]
tuple([1, 2])           # (1, 2)
set([1, 1, 2])          # {1, 2}
dict([("a", 1), ("b", 2)])
dict(zip(keys, vals))
```

| From → To | Common pattern |
| :--- | :--- |
| list → unique | `list(dict.fromkeys(xs))` keep order |
| pairs → dict | `dict(pairs)` or `{k: v for ...}` |
| dict → keys list | `list(d)` or `list(d.keys())` |
| two lists → dict | `dict(zip(ks, vs))` |

---

## 7. Copying: shallow vs deep

Assignment shares the container. `.copy()` / `list(xs)` / `xs[:]` make a **shallow** copy: new outer container, same element objects.

![Shallow copy](./assets/07-lists-tuples-sets-dicts/shallow-copy.svg)

```python
import copy

a = [[1], 2]
b = a.copy()           # shallow
c = copy.deepcopy(a)   # full independent tree

a[0].append(9)
# b[0] is also [1, 9]
# c[0] still [1]
```

| Goal | Tool |
| :--- | :--- |
| Same object | `b = a` |
| New outer, shared inners | `a.copy()`, `list(a)`, `dict(a)`, `set(a)` |
| Independent nest | `copy.deepcopy(a)` |

---

## 8. Comprehensions for all four

```python
xs = [x * 2 for x in range(5)]           # list
ts = tuple(x * 2 for x in range(5))      # tuple (via generator)
ss = {x % 3 for x in range(10)}          # set
dd = {c: ord(c) for c in "ab"}           # dict
```

Dict and set comps use `{}` with different insides: `{k: v ...}` vs `{x ...}`.

---

## 9. Mini example — inventory sketch

Scenario: track stock SKUs, unique tags, and a fixed warehouse coordinate.

```python
# list — ordered line items you edit
cart = ["sku-1", "sku-2", "sku-1"]

# set — unique tags for search
tags = {"electronics", "clearance"}
tags.add("gift")

# dict — sku → quantity
stock = {"sku-1": 10, "sku-2": 3}
stock["sku-1"] -= 1

# tuple — immutable location record (and hashable key)
warehouse = ("A", 12)          # aisle, bin
locations = {warehouse: stock}

print(cart.count("sku-1"))     # 2
print("gift" in tags)          # True
print(locations[("A", 12)]["sku-2"])  # 3
```

---

## 10. Common gotchas

| Pitfall | What happens | Fix |
| :--- | :--- | :--- |
| `{}` for empty set | Creates empty **dict** | `set()` |
| Mutating list while iterating | Skipped / wrong elements | New list or iterate copy |
| `list` as dict key | `TypeError: unhashable` | `tuple(...)` if possible |
| Shallow copy surprise | Nested mutate shared | `deepcopy` or redesign |
| `dict` iteration + insert | RuntimeError if size changes | Build a new dict |
| Using list for membership in hot loop | O(n) each check | Convert to `set` once |
| Assuming set order | No index; order not semantic | Don’t rely on order for logic |
| Tuple “immutable” myth | Inner list still mutates | Only slots are fixed |

> [!WARNING]
> Default argument `def f(xs=[])` shares one list across calls. Use `None` and create inside. Same trap for `{}` and `set()`. See [04](./04-basic-syntax-and-data-types.md).

```python
# membership upgrade
allowed = set(allowed_list)
if user_id in allowed:
    ...
```

---

## 11. Complexity cheat sheet

| Structure | Index | Membership | Insert typical |
| :--- | :--- | :--- | :--- |
| `list` | O(1) | O(n) | end O(1) amort.; middle O(n) |
| `tuple` | O(1) | O(n) | n/a (immutable) |
| `set` | n/a | O(1) avg | O(1) avg |
| `dict` | by key O(1) avg | key O(1) avg | O(1) avg |

“Average” for hash structures assumes good hash distribution; worst case can degrade (rare in normal Python use).

---

## Sources

- [Python tutorial — Data structures](https://docs.python.org/3/tutorial/datastructures.html)
- [Built-in types](https://docs.python.org/3/library/stdtypes.html)
- [TimeComplexity (Python wiki)](https://wiki.python.org/moin/TimeComplexity)
- [dict order guarantee (3.7)](https://docs.python.org/3/library/stdtypes.html#dict)

## Related

- [Basic Syntax and Data Types](./04-basic-syntax-and-data-types.md)
- [Conditionals and Loops](./05-conditionals-and-loops.md)
- [Functions and Classes](./08-functions-and-classes.md)
- [Understanding the Language](./06-understanding-the-language.md)
- [Python Road Map](./01-python-road-map.md)
