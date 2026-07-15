---
title: "Container With Most Water"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, two-pointers, greedy, arrays, neetcode]
aliases: []
---

# Container With Most Water

[toc]

> **TL;DR:** Start with the widest container. At each step, compute area, then move the pointer at the shorter wall because the shorter wall is the limiting height.

## Vocabulary

**Container**

Two vertical bars and the horizontal distance between them.

**Area**

The amount of water held by two bars: width times the shorter height.

**Limiting wall**

The shorter of the two bars. It determines the container height.

## Underlying Concept

The brute force solution checks every pair. The two-pointer solution starts with the widest pair and safely discards the weaker side.

```text
area = (right - left) * min(height[left], height[right])
```

The width shrinks every time a pointer moves. So to possibly improve area, we need a taller limiting wall.

> [!IMPORTANT]
> Move the shorter wall. Moving the taller wall cannot help while the shorter wall still limits the height and the width gets smaller.

## Why Move The Shorter Pointer

Suppose `height[left] < height[right]`. The current container is limited by `height[left]`.

If we keep `left` and move `right` inward, the width shrinks and the height is still at most `height[left]`. That cannot beat the current area.

So index `left` can be safely discarded.

```python
if height[left] < height[right]:
    left += 1
else:
    right -= 1
```

This is the greedy proof: discard the side that cannot form a better future container.

## Full Python Solution

The algorithm checks one container per pointer movement.

```python
from typing import List


class Solution:
    def maxArea(self, height: List[int]) -> int:
        left = 0
        right = len(height) - 1
        best = 0

        while left < right:
            width = right - left
            current_height = min(height[left], height[right])
            best = max(best, width * current_height)

            if height[left] < height[right]:
                left += 1
            else:
                right -= 1

        return best
```

The maximum is stored in `best` because the optimal pair might appear before the pointers meet.

## Dry Run

Use `height = [1, 7, 2, 5, 4, 7, 3, 6]`.

| Step | `left` | `right` | Heights | Area | Move |
| ---: | ---: | ---: | :--- | ---: | :--- |
| 1 | `0` | `7` | `1, 6` | `7` | move left |
| 2 | `1` | `7` | `7, 6` | `36` | move right |
| 3 | `1` | `6` | `7, 3` | `15` | move right |
| 4 | `1` | `5` | `7, 7` | `28` | move right |

The best area seen is `36`.

## Edge Cases

These cases explain the formula and pointer move.

| Case | Example | Result |
| :--- | :--- | ---: |
| Two bars | `[1, 2]` | `1` |
| Equal heights | `[2, 2, 2]` | `4` |
| Tall bars far apart | `[5, 1, 1, 5]` | `15` |
| Zero height | `[0, 2, 0, 3]` | `4` |

> [!WARNING]
> Do not move both pointers every time. Only the shorter side has been proven safe to discard.

## Complexity

The pointers only move inward.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(n)` | Each index is visited at most once by a pointer. |
| Space | `O(1)` | Only pointers and best area are stored. |

## Interview Questions and Answers

**Q: What is the area formula?**  
A: `(right - left) * min(height[left], height[right])`.

**Q: Why move the shorter wall?**  
A: It limits the height. Keeping it while shrinking width cannot produce a bigger area.

**Q: Why start at the ends?**  
A: The ends give the maximum initial width, then the algorithm trades width for a chance at taller walls.

**Q: What should you say out loud in an interview?**  
A: "I compute the current area, update the best answer, then discard the shorter wall because it cannot make a better future container at smaller width."

## Sources

- [NeetCode: Container With Most Water](https://neetcode.io/problems/max-water-container/question?list=neetcode150)
- Problem request from user on 2026-07-15.

## Related

- [Two Pointers](../../02-two-pointers.md)
- [Trapping Rain Water](./trapping-rain-water.md)
