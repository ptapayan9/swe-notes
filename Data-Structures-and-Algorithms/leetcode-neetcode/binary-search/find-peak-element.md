---
title: "Find Peak Element"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, arrays, neetcode]
aliases: []
---

# Find Peak Element

[toc]

> **TL;DR:** Read the slope. If `nums[mid] < nums[mid + 1]`, you are climbing, so a peak must exist on the right. Otherwise, a peak exists at `mid` or to the left.

## Vocabulary

**Peak**

An index whose value is greater than its neighbors. The problem treats values outside the array as negative infinity.

**Slope**

The direction between adjacent values. If `nums[mid] < nums[mid + 1]`, the slope goes up.

**Candidate-preserving search**

A binary search where `mid` might still be the answer. That is why one branch uses `right = mid`.

## Underlying Concept

This is binary search without a target. Instead of asking "is `nums[mid]` equal to target?", we ask which direction contains a guaranteed peak.

```text
nums = [1, 2, 3, 1]

1 -> 2 -> 3 <- 1
          ^
          peak
```

If the slope goes up from `mid` to `mid + 1`, there must be a peak on the right. If the slope goes down or stays not-up, there must be a peak at `mid` or on the left.

> [!IMPORTANT]
> We do not need the array to be sorted. We only need the local slope comparison `nums[mid] < nums[mid + 1]`.

## Why Compare Mid to Mid Plus One

The comparison reads whether we are going uphill or downhill.

```python
if nums[mid] < nums[mid + 1]:
    left = mid + 1
else:
    right = mid
```

If we are going uphill, then a peak must eventually appear to the right. It might be `mid + 1`, or it might be later.

If we are not going uphill, then `mid` could already be a peak, or a peak exists to the left.

## Full Python Solution

Use `while left < right` so `mid + 1` is always inside the current window. When the loop ends, `left` and `right` point to a peak.

```python
from typing import List


class Solution:
    def findPeakElement(self, nums: List[int]) -> int:
        left = 0
        right = len(nums) - 1

        while left < right:
            mid = (left + right) // 2

            if nums[mid] < nums[mid + 1]:
                left = mid + 1
            else:
                right = mid

        return left
```

This returns any valid peak. The problem does not require the first peak or the highest peak.

## Dry Run

Use `nums = [1, 2, 3, 1]`.

| Step | `left` | `right` | `mid` | Compare | Decision |
| ---: | ---: | ---: | ---: | :--- | :--- |
| 1 | `0` | `3` | `1` | `2 < 3` | uphill, move `left = 2` |
| 2 | `2` | `3` | `2` | `3 < 1` is false | peak at mid or left, move `right = 2` |

Now `left == right == 2`, so return index `2`.

## Why This Is Safe

The search never throws away all peaks. Each branch keeps a side where at least one peak must exist.

| Slope check | What it proves | Move |
| :--- | :--- | :--- |
| `nums[mid] < nums[mid + 1]` | A peak exists to the right. | `left = mid + 1` |
| `nums[mid] > nums[mid + 1]` | A peak exists at `mid` or to the left. | `right = mid` |

The problem usually gives adjacent unequal values. With equal adjacent values, the proof needs extra care, but the standard LeetCode version avoids that ambiguity.

## Edge Cases

These cases explain the loop shape.

| Case | Example | Result |
| :--- | :--- | :--- |
| One element | `[1]` | index `0` |
| Peak at start | `[3, 2, 1]` | index `0` |
| Peak at end | `[1, 2, 3]` | index `2` |
| Multiple peaks | `[1, 3, 2, 4, 1]` | either index `1` or `3` |
| Valley then peak | `[2, 1, 3]` | index `0` or `2` |

> [!WARNING]
> Do not use `while left <= right` with direct `nums[mid + 1]` access. The `while left < right` pattern keeps `mid + 1` safe.

## Complexity

Each step cuts the search window roughly in half.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log n)` | Binary search by slope. |
| Space | `O(1)` | Only pointers are stored. |

## Interview Questions and Answers

**Q: What are we binary searching for?**  
A: A peak index, found by reading the local slope around `mid`.

**Q: Why does uphill mean go right?**  
A: If `nums[mid] < nums[mid + 1]`, moving right is climbing. A peak must eventually occur on that side.

**Q: Why does the else branch use `right = mid`?**  
A: `mid` might itself be a peak, so we keep it as a candidate.

**Q: Does the array need to be sorted?**  
A: No. This binary search uses local slope, not global sorted order.

**Q: What should you say out loud in an interview?**  
A: "I compare mid with mid plus one. If the slope rises, I move right; otherwise I keep mid and move left."

## Sources

- [LeetCode: Find Peak Element](https://leetcode.com/problems/find-peak-element/)
- Problem list provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Find Minimum in Rotated Sorted Array](./find-minimum-in-rotated-sorted-array.md)
