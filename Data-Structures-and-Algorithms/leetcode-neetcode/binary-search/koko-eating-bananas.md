---
title: "Koko Eating Bananas"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, binary-search-on-answer, neetcode]
aliases: []
---

# Koko Eating Bananas

[toc]

> **TL;DR:** This is binary search on the answer, not binary search inside `piles`. Try an eating speed `k`, compute how many hours that speed needs, then shrink toward the smallest speed that still finishes within `h`.

## Vocabulary

**Eating rate**

The integer speed `k`, measured in bananas per hour.

**Feasible speed**

A speed that lets Koko finish every pile within `h` hours.

**Monotonic predicate**

A true-or-false check where once a speed works, every larger speed also works. For this problem, `can_finish(k)` changes from `False` to `True` as `k` increases.

**Ceiling division**

Division that rounds up to the next whole number. A pile of `3` bananas at speed `2` takes `2` hours, not `1.5`, because Koko eats in whole hours.

## Problem Pattern

The piles are not sorted, and sorting them does not solve the problem. The sorted structure is the possible answer range.

Koko's speed can be:

```text
1, 2, 3, ..., max(piles)
```

If speed `k` is too slow, every smaller speed is also too slow. If speed `k` works, every larger speed also works.

```text
k values:        1    2    3    4    5    ...
can finish?   False True True True True ...
                    ^
                    smallest valid k
```

That `False ... True ...` shape is exactly what binary search can find.

> [!IMPORTANT]
> Binary search works here because feasibility is monotonic. We are not searching for a pile. We are searching for the first speed where `can_finish(k)` becomes true.

## Why Each Pile Uses Ceiling Division

Each hour, Koko chooses one pile and eats up to `k` bananas from that pile. If the pile has fewer than `k` bananas, she finishes the pile but cannot start another pile in that same hour.

That means each pile's time is rounded up.

```text
pile = 4, k = 2 -> 2 hours
pile = 3, k = 2 -> 2 hours
pile = 1, k = 2 -> 1 hour
```

In Python, compute that without floating point:

```python
hours += (pile + k - 1) // k
```

This is the integer version of rounding up. It avoids `math.ceil(pile / k)` and stays exact for large values.

### Why the Formula Rounds Up

Normal integer division with `//` rounds down. That is not enough for this problem, because any leftover bananas still cost one more full hour.

```text
pile = 5, k = 2

5 // 2 = 2

But Koko needs:
hour 1 -> eat 2 bananas
hour 2 -> eat 2 bananas
hour 3 -> eat 1 banana

actual hours = 3
```

The formula adds `k - 1` before dividing:

```python
(pile + k - 1) // k
```

That extra `k - 1` only pushes the result into the next integer bucket when there is a remainder.

| `pile` | `k` | Normal `pile // k` | Ceiling formula | Hours needed |
| ---: | ---: | ---: | :--- | ---: |
| `4` | `2` | `2` | `(4 + 2 - 1) // 2 = 5 // 2` | `2` |
| `3` | `2` | `1` | `(3 + 2 - 1) // 2 = 4 // 2` | `2` |
| `5` | `2` | `2` | `(5 + 2 - 1) // 2 = 6 // 2` | `3` |
| `10` | `3` | `3` | `(10 + 3 - 1) // 3 = 12 // 3` | `4` |

Another way to say it:

- `pile // k` counts the full `k`-banana hours.
- If `pile % k` leaves anything over, Koko needs one more hour.
- `(pile + k - 1) // k` combines both steps in one integer expression.

```python
full_hours = pile // k
has_leftover = pile % k != 0

if has_leftover:
    full_hours += 1
```

The compact formula is the same idea, just shorter.

## Checking One Speed

The helper function answers one question: "At speed `k`, can Koko finish within `h` hours?"

```python
def can_finish(k):
    hours = 0

    for pile in piles:
        hours += (pile + k - 1) // k

    return hours <= h
```

If `hours <= h`, speed `k` is fast enough. If `hours > h`, speed `k` is too slow.

## Choosing Left and Right

The slowest possible useful speed is `1`. A speed of `0` would never eat bananas, so it is not valid.

The fastest speed we ever need to try is `max(piles)`.

```python
left = 1
right = max(piles)
```

At `k = max(piles)`, each pile takes at most one hour. Since the constraints guarantee `h >= len(piles)`, this speed always works.

### Why Left Starts at 1

`left` is the smallest eating speed we are willing to test. Koko must eat at least one banana per hour, so the smallest valid `k` is `1`.

```python
left = 1
```

Starting at `0` breaks the meaning of the problem and can crash the formula:

```python
(pile + k - 1) // k
```

If `k = 0`, this becomes division by zero. More importantly, a speed of zero bananas per hour could never finish any non-empty pile.

### Why Right Is Max Piles

`right` is the largest eating speed we need to consider. The largest pile is the highest useful speed because it lets Koko finish every pile in at most one hour.

```python
right = max(piles)
```

Example:

```text
piles = [25, 10, 23, 4]
max(piles) = 25

At k = 25:
25 -> 1 hour
10 -> 1 hour
23 -> 1 hour
4  -> 1 hour

total = 4 hours
```

Since the problem guarantees `h >= len(piles)`, one hour per pile is always fast enough.

Do not use `len(piles)` for `right`. `len(piles)` is the number of piles, not an eating speed.

```text
piles = [25, 10, 23, 4]
len(piles) = 4

But the answer can be 25.
```

If `right = len(piles)`, the binary search might never even consider the real answer.

> [!TIP]
> You can sometimes start `left` at `ceil(sum(piles) / h)` for a tighter lower bound, but `left = 1` is simpler and still passes easily.

## Binary Search Logic

We want the minimum valid speed. When `mid` works, keep it as a candidate and search left for a smaller working speed.

When `mid` does not work, discard it and every smaller speed by moving rightward.

```python
while left < right:
    mid = (left + right) // 2

    if can_finish(mid):
        right = mid
    else:
        left = mid + 1

return left
```

This uses `while left < right` because we are narrowing to the first valid speed. The answer remains inside the window, and the loop stops when `left` and `right` meet.

## Full Python Solution

This is the standard "first true" binary search pattern. The predicate is `can_finish(k)`.

```python
from typing import List


class Solution:
    def minEatingSpeed(self, piles: List[int], h: int) -> int:
        def can_finish(k: int) -> bool:
            hours = 0

            for pile in piles:
                hours += (pile + k - 1) // k

            return hours <= h

        left = 1
        right = max(piles)

        while left < right:
            mid = (left + right) // 2

            if can_finish(mid):
                right = mid
            else:
                left = mid + 1

        return left
```

You can also write the helper inline, but naming it `can_finish` makes the binary search easier to read.

## Dry Run

Use `piles = [1, 4, 3, 2]` and `h = 9`. The answer is `2`.

First, see why `2` works:

```text
k = 2

pile 1 -> 1 hour
pile 4 -> 2 hours
pile 3 -> 2 hours
pile 2 -> 1 hour

total = 6 hours
```

Now see how binary search finds that speed.

| Step | `left` | `right` | `mid` | Hours needed | Decision |
| ---: | ---: | ---: | ---: | ---: | :--- |
| 1 | `1` | `4` | `2` | `6` | `6 <= 9`, speed works, keep it with `right = 2` |
| 2 | `1` | `2` | `1` | `10` | `10 > 9`, too slow, move `left = 2` |

Now `left == right == 2`, so the minimum valid speed is `2`.

## Another Example

Use `piles = [25, 10, 23, 4]` and `h = 4`. Since there are four piles and only four hours, each pile must finish in exactly one hour.

That means Koko needs to eat the largest pile in one hour.

```text
max(piles) = 25
answer = 25
```

Any smaller speed would make the pile of `25` take at least two hours, which would exceed the four-hour limit.

## Edge Cases

These cases help separate binary search on values from binary search on indexes.

| Case | Example | Why it matters |
| :--- | :--- | :--- |
| One pile | `piles = [10]`, `h = 2` | Answer is `5`, using ceiling division. |
| `h == len(piles)` | `piles = [25, 10, 23, 4]`, `h = 4` | Must eat each pile in one hour, so answer is `max(piles)`. |
| Plenty of hours | `piles = [1, 4, 3, 2]`, `h = 100` | Minimum speed can be `1`. |
| Large pile values | `piles[i]` up to `1_000_000_000` | Use integer arithmetic; avoid simulating bananas one by one. |
| Unsorted piles | `[30, 11, 23, 4, 20]` | The pile order does not matter for the feasibility check. |

> [!WARNING]
> Do not try to binary search indexes in `piles`. The array is not sorted by usefulness. The sorted search space is the possible speed `k`.

## Complexity

Each binary-search step scans every pile once to count hours. The number of speed guesses is logarithmic in the largest pile.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(n log m)` | `n` is the number of piles, `m = max(piles)`. |
| Space | `O(1)` | Only counters and pointers are stored. |

## Interview Questions and Answers

**Q: What are we binary searching over?**  
A: We binary search over possible speeds `k`, from `1` to `max(piles)`.

**Q: Why is this search space sorted?**  
A: If a speed works, every faster speed also works. If a speed is too slow, every slower speed is also too slow.

**Q: Why is the upper bound `max(piles)`?**  
A: At that speed, every pile takes at most one hour. Since `h >= len(piles)`, that speed is guaranteed to finish in time.

**Q: Why do we use ceiling division for each pile?**  
A: Koko cannot start a new pile in the leftover part of an hour. Even a partially eaten final hour still counts as a full hour.

**Q: Why does a working `mid` move `right = mid` instead of returning?**  
A: We need the minimum working speed. `mid` works, but a smaller speed might also work, so keep `mid` and search left.

**Q: What should you say out loud in an interview?**  
A: "I define `can_finish(k)` as hours needed at speed `k` being at most `h`. Because that predicate flips from false to true as `k` increases, I binary search for the first true speed."

## Sources

- [LeetCode: Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)
- Problem statement provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Find Minimum in Rotated Sorted Array](./find-minimum-in-rotated-sorted-array.md)
