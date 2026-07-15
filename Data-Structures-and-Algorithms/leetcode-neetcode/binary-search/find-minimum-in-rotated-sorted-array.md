---
title: "Find Minimum in Rotated Sorted Array"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, rotated-array, neetcode]
aliases: []
---

# Find Minimum in Rotated Sorted Array

[toc]

> **TL;DR:** The minimum is the rotation pivot, where the sorted array "wraps around." Compare `nums[mid]` to `nums[right]`: if mid is bigger, the minimum is to the right; otherwise mid might be the minimum, so keep it by moving `right = mid`.

## Vocabulary

**Rotated sorted array**

An ascending sorted array shifted so that some suffix moves to the front. Example: `[1, 2, 3, 4, 5, 6]` can become `[4, 5, 6, 1, 2, 3]`.

**Pivot**

The index where the array wraps from large values back to small values. In `[4, 5, 6, 1, 2, 3]`, the pivot is index `3`, where the value is `1`.

**Search window**

The current inclusive range from `left` to `right` where the minimum could still exist.

**Candidate-preserving binary search**

A binary search pattern where `mid` may still be the answer, so one branch uses `right = mid` instead of `right = mid - 1`.

## Problem Pattern

The original array was sorted in ascending order, then rotated. Since all elements are unique, there is one clean minimum.

The minimum is easy to find in linear time, but the goal is `O(log n)`. That means every step must eliminate about half of the remaining window.

```text
nums = [3, 4, 5, 6, 1, 2]

sorted pieces:
[3, 4, 5, 6] [1, 2]
             ^
             minimum / pivot
```

The trick is to use the rightmost value as an anchor. Comparing `nums[mid]` to `nums[right]` tells us which side contains the pivot.

## Why Compare Mid to Right

The right side is a stable reference point. If `nums[mid] > nums[right]`, then `mid` is in the larger left sorted piece and the array must wrap somewhere to the right.

If `nums[mid] < nums[right]`, then the range from `mid` to `right` is sorted normally. The minimum cannot be after `mid`; it is either at `mid` or somewhere to the left.

```text
Case 1: nums[mid] > nums[right]

[3, 4, 5, 6, 1, 2]
       M        R

5 > 2, so the drop from high to low is on the right.
Move left to mid + 1.

Case 2: nums[mid] < nums[right]

[6, 1, 2, 3, 4, 5]
       M        R

2 < 5, so mid-to-right is sorted.
The minimum is at mid or to the left.
Move right to mid.
```

> [!IMPORTANT]
> Use `right = mid`, not `right = mid - 1`, when `nums[mid] < nums[right]`. `mid` could be the minimum, so throwing it away can lose the answer.

## Binary Search Logic

This is not exact search for a target. We are shrinking toward the smallest value until `left` and `right` point to the same index.

Because we sometimes keep `mid` as a possible answer, the loop condition is `while left < right`. When the loop ends, there is one candidate left.

```python
left = 0
right = len(nums) - 1

while left < right:
    mid = (left + right) // 2

    if nums[mid] > nums[right]:
        left = mid + 1
    else:
        right = mid

return nums[left]
```

The two branches mean:

| Condition | What it proves | Move |
| :--- | :--- | :--- |
| `nums[mid] > nums[right]` | The minimum is strictly after `mid`. | `left = mid + 1` |
| `nums[mid] < nums[right]` | `mid` could be the minimum, or the minimum is left of `mid`. | `right = mid` |

Since the problem says all values are unique, there is no duplicate ambiguity. If duplicates were allowed, `nums[mid] == nums[right]` would require special handling.

## Full Python Solution

The solution keeps the current search window inclusive. Each iteration asks whether the pivot is to the right of `mid` or at-or-left of `mid`.

```python
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        left = 0
        right = len(nums) - 1

        while left < right:
            mid = (left + right) // 2

            if nums[mid] > nums[right]:
                left = mid + 1
            else:
                right = mid

        return nums[left]
```

The constraints guarantee at least one element. If `nums` has one value, the loop never runs and the function returns `nums[0]`.

## Dry Run

Use `nums = [3, 4, 5, 6, 1, 2]`. The minimum is `1`, but the algorithm discovers that by asking where the rotation drop must be.

```text
index:  0  1  2  3  4  5
value: [3, 4, 5, 6, 1, 2]
```

The search keeps cutting the window until one index remains.

| Step | `left` | `right` | `mid` | `nums[mid]` | `nums[right]` | Decision |
| ---: | ---: | ---: | ---: | ---: | ---: | :--- |
| 1 | `0` | `5` | `2` | `5` | `2` | `5 > 2`, min is right of mid, `left = 3` |
| 2 | `3` | `5` | `4` | `1` | `2` | `1 < 2`, keep mid, `right = 4` |
| 3 | `3` | `4` | `3` | `6` | `1` | `6 > 1`, min is right of mid, `left = 4` |

Now `left == right == 4`, so return `nums[4]`, which is `1`.

## Edge Cases

These cases are where the pointer updates matter most.

| Case | Example | Why it matters |
| :--- | :--- | :--- |
| One element | `[7]` | Loop skips and returns the only value. |
| Not visually rotated | `[4, 5, 6, 7]` | This can happen after rotating `n` times; answer is the first value. |
| Pivot near the end | `[2, 3, 4, 5, 1]` | Confirms `left = mid + 1` can jump toward the final index. |
| Pivot near the beginning | `[5, 1, 2, 3, 4]` | Confirms `right = mid` preserves the candidate side. |
| Negative values | `[0, 3, 8, -4, -1]` | Values can be negative; ordering logic still works. |

> [!WARNING]
> Do not use `right = mid - 1` in the "mid is smaller than right" branch. In `[4, 5, 1, 2, 3]`, `mid` can land directly on `1`, and discarding it would skip the answer.

## Why Not Compare to Left

Comparing to `nums[left]` can work in some variants, but it is easier to make mistakes because `left` may itself be the minimum. The right edge gives a clearer test for whether the drop is on the right side.

With `nums[right]`:

- `nums[mid] > nums[right]` means the right side contains the wrap.
- `nums[mid] < nums[right]` means mid-to-right is sorted, so the minimum is not after mid.

That gives a small, stable template.

## Complexity

Each loop removes about half of the remaining search window. The algorithm stores only a few pointers.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log n)` | Binary search over the array. |
| Space | `O(1)` | No extra data structure is created. |

## Interview Questions and Answers

**Q: What are we searching for if there is no target?**  
A: We are searching for the pivot, which is the smallest value and the point where the rotated array wraps.

**Q: Why does `nums[mid] > nums[right]` mean the minimum is on the right?**  
A: If mid is bigger than the rightmost value, the array must drop somewhere after mid. That drop contains the minimum.

**Q: Why does the other branch use `right = mid`?**  
A: If `nums[mid] < nums[right]`, mid-to-right is sorted. The minimum could be exactly at mid, so keep mid as a candidate.

**Q: Why is the loop `while left < right` instead of `while left <= right`?**  
A: We are narrowing to one remaining candidate, not checking for an exact target. Once `left == right`, that index is the answer.

**Q: What changes if duplicates are allowed?**  
A: If `nums[mid] == nums[right]`, you cannot prove which side contains the minimum. A common fix is `right -= 1`, but that can degrade to `O(n)` in the worst case.

**Q: What should you say out loud in an interview?**  
A: "I compare mid to the right edge. If mid is larger, the pivot is to the right. Otherwise, mid might be the pivot, so I keep it and move right to mid."

## Sources

- [LeetCode: Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
- Problem statement provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Search in a 2D Matrix](./search-in-a-2d-matrix.md)
