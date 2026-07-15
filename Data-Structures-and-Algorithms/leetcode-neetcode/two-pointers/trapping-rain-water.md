---
title: "Trapping Rain Water"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, two-pointers, arrays, neetcode]
aliases: []
---

# Trapping Rain Water

[toc]

> **TL;DR:** Water above an index depends on the shorter of the tallest wall to its left and tallest wall to its right. Two pointers can track those walls from both sides in one pass.

## Vocabulary

**Elevation map**

An array where each value is the height of a bar with width `1`.

**Left max**

The tallest bar seen so far from the left side.

**Right max**

The tallest bar seen so far from the right side.

**Trapped water**

At one index, the water level is limited by the shorter side wall.

## Underlying Concept

At each index, trapped water is:

```text
min(tallest left wall, tallest right wall) - current height
```

If that value is negative, the index traps no water.

The two-pointer version avoids building prefix and suffix arrays. It keeps `left_max` and `right_max` while moving inward.

> [!IMPORTANT]
> Process the side with the smaller current max. That smaller max is the limiting side, so water on that side is knowable now.

## Why The Smaller Max Side Moves

If `left_max < right_max`, then the left side is the bottleneck. We already know there is a right wall tall enough to support water up to `left_max`.

So the water at `left` is:

```text
left_max - height[left]
```

If `right_max <= left_max`, process the right side for the same reason.

This is the key difference from Container With Most Water: here we move based on the smaller current max, not just the shorter current bar.

## Full Python Solution

This version runs in one pass and uses constant extra space.

```python
from typing import List


class Solution:
    def trap(self, height: List[int]) -> int:
        if not height:
            return 0

        left = 0
        right = len(height) - 1
        left_max = height[left]
        right_max = height[right]
        water = 0

        while left < right:
            if left_max < right_max:
                left += 1
                left_max = max(left_max, height[left])
                water += left_max - height[left]
            else:
                right -= 1
                right_max = max(right_max, height[right])
                water += right_max - height[right]

        return water
```

Updating the max before adding water keeps the subtraction non-negative.

## Dry Run

Use `height = [0, 2, 0, 3, 1, 0, 1, 3, 2, 1]`.

The left side starts with `left_max = 0`, and the right side starts with `right_max = 1`.

| Step | Side processed | Height | Known max | Water added |
| ---: | :--- | ---: | ---: | ---: |
| 1 | left | `2` | `2` | `0` |
| 2 | right | `2` | `2` | `0` |
| 3 | left | `0` | `2` | `2` |
| 4 | left | `3` | `3` | `0` |
| 5 | right | `3` | `3` | `0` |

Continuing this process gives total trapped water `9`.

## Prefix And Suffix Alternative

A more visual approach builds two arrays first. `prefix[i]` stores the tallest wall from the left through `i`, and `suffix[i]` stores the tallest wall from the right through `i`.

Then each index contributes:

```text
min(prefix[i], suffix[i]) - height[i]
```

That solution is easier to reason about at first, but it uses `O(n)` extra space. The two-pointer version compresses the same idea into two running maximums.

## Edge Cases

These cases clarify when water can and cannot be trapped.

| Case | Example | Result |
| :--- | :--- | ---: |
| Empty or one bar | `[]`, `[1]` | `0` |
| Strictly increasing | `[1, 2, 3]` | `0` |
| Strictly decreasing | `[3, 2, 1]` | `0` |
| Simple bowl | `[2, 0, 2]` | `2` |
| Multiple valleys | `[0, 2, 0, 3, 1, 0, 1, 3, 2, 1]` | `9` |

> [!WARNING]
> Do not add water using only the current left and right heights. You need the best wall seen so far from each side.

## Complexity

The two pointers scan the array once.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(n)` | Each pointer moves inward. |
| Space | `O(1)` | Only max values, pointers, and total water are stored. |

## Interview Questions and Answers

**Q: What determines water above one bar?**  
A: The shorter of the tallest wall on the left and tallest wall on the right.

**Q: Why process the side with smaller max?**  
A: That side is the limiting wall, so its trapped water can be computed safely.

**Q: How is this different from Container With Most Water?**  
A: Container maximizes pair area. Trapping Rain Water sums water over many positions using running boundary walls.

**Q: What should you say out loud in an interview?**  
A: "I track the best wall from both sides. The smaller max is the limiting boundary, so I process that side and add the trapped water there."

## Sources

- [NeetCode: Trapping Rain Water](https://neetcode.io/problems/trapping-rain-water/question?list=neetcode150)
- Problem request from user on 2026-07-15.

## Related

- [Two Pointers](../../02-two-pointers.md)
- [Container With Most Water](./container-with-most-water.md)
