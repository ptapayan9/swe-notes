---
title: "Search in Rotated Sorted Array"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, rotated-array, neetcode]
aliases: []
---

# Search in Rotated Sorted Array

[toc]

> **TL;DR:** A rotated sorted array is made of two sorted pieces. At every binary-search step, at least one side of `mid` is sorted; identify that sorted side, check whether the target belongs inside it, then discard the other side.

## Vocabulary

**Rotated sorted array**

An ascending sorted array shifted so that a suffix moves to the front. Example: `[1, 2, 3, 4, 5, 6]` can become `[4, 5, 6, 1, 2, 3]`.

**Pivot**

The place where the array wraps from large values back to small values. In `[4, 5, 6, 1, 2, 3]`, the pivot value is `1`.

**Sorted half**

The side of the current search window that is still in normal ascending order. In each iteration, either `left..mid` or `mid..right` is sorted.

**Target range check**

The test that asks whether `target` fits inside the sorted half's endpoints.

## Problem Pattern

Normal binary search works because the whole array is sorted. A rotated sorted array is not globally sorted, but it still has structure.

Inside any current search window, one side of `mid` must be sorted. Use that sorted half as your proof for which side can be discarded.

```text
nums = [4, 5, 6, 7, 0, 1, 2]
target = 0

left side around mid:  [4, 5, 6, 7]
right side around mid: [0, 1, 2]
```

The target search is still `O(log n)` because every iteration removes about half of the current window.

> [!IMPORTANT]
> This problem assumes all values are unique. If duplicates are allowed, the sorted-half decision can become ambiguous.

## The Key Question

After checking whether `nums[mid] == target`, ask one question: "Which half is sorted?"

Your draft was heading toward this line:

```python
if nums[mid] > nums[left]:
```

The safer version is:

```python
if nums[left] <= nums[mid]:
```

Use `<=` because when `left == mid`, the left half has one element and is still sorted. Using only `>` can misclassify small windows.

```text
nums = [3, 1], target = 1

left = 0, right = 1, mid = 0
nums[mid] == nums[left]

The left side is one element: [3].
That is sorted, so the test should treat it as sorted.
```

## If the Left Half Is Sorted

When `nums[left] <= nums[mid]`, the values from `left` to `mid` are in ascending order. **That lets us ask whether the target fits between the two endpoints.**

```python
if nums[left] <= nums[mid]:
    if nums[left] <= target < nums[mid]:
        right = mid - 1
    else:
        left = mid + 1
```

The range check uses `nums[left] <= target < nums[mid]`. It includes the left endpoint and excludes `mid` because `mid` was already checked.

If the target fits in the sorted left half, move `right` leftward. Otherwise, discard the left half and search the right side.

## If the Right Half Is Sorted

If the left half is not sorted, then the right half must be sorted. That means the pivot is somewhere on the left side.

```python
else:
    if nums[mid] < target <= nums[right]:
        left = mid + 1
    else:
        right = mid - 1
```

The range check uses `nums[mid] < target <= nums[right]`. It excludes `mid` because `mid` was already checked and includes the right endpoint.

If the target fits in the sorted right half, move `left` rightward. Otherwise, discard the right half and search the left side.

## Full Python Solution

This is exact binary search. Every time we check `mid`, either we return it or discard it with `mid + 1` or `mid - 1`.

```python
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = (left + right) // 2

            if nums[mid] == target:
                return mid

            # Left half is sorted.
            if nums[left] <= nums[mid]:
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1
            # Right half is sorted.
            else:
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1

        return -1
```

The most important habit is separating the logic into two questions:

1. Which half is sorted?
2. Does the target belong inside that sorted half?

## Dry Run

Use `nums = [4, 5, 6, 7, 0, 1, 2]` and `target = 0`. The answer is index `4`.

```text
index:  0  1  2  3  4  5  6
value: [4, 5, 6, 7, 0, 1, 2]
```

The algorithm first finds that the left side is sorted, then discards it because the target does not fit there.

| Step | `left` | `right` | `mid` | `nums[mid]` | Sorted half | Decision |
| ---: | ---: | ---: | ---: | ---: | :--- | :--- |
| 1 | `0` | `6` | `3` | `7` | Left: `[4, 5, 6, 7]` | `0` is not in `[4, 7)`, so `left = 4` |
| 2 | `4` | `6` | `5` | `1` | Left: `[0, 1]` | `0` is in `[0, 1)`, so `right = 4` |
| 3 | `4` | `4` | `4` | `0` | Found | Return `4` |

The last row happens because the loop is `while left <= right`. When one candidate remains, it still needs to be checked.

## Edge Cases

These cases expose most wrong branch conditions.

| Case | Example | Why it matters |
| :--- | :--- | :--- |
| One element, found | `[1]`, target `1` | The first equality check should return `0`. |
| One element, missing | `[1]`, target `0` | The loop ends and returns `-1`. |
| Two elements rotated | `[3, 1]`, target `1` | Requires `nums[left] <= nums[mid]`, not only `nums[mid] > nums[left]`. |
| Not visually rotated | `[1, 2, 3, 4]`, target `3` | Rotation by `n` gives the original sorted array. |
| Target in left sorted half | `[4, 5, 6, 7, 0, 1, 2]`, target `5` | Confirms the left range check. |
| Target in right sorted half | `[4, 5, 6, 7, 0, 1, 2]`, target `1` | Confirms the right range check. |
| Target missing | `[4, 5, 6, 7, 0, 1, 2]`, target `3` | Search should exhaust the window. |

> [!WARNING]
> Do not compare only `nums[mid]` to `target` to decide direction. Rotation breaks the simple "smaller means go right, larger means go left" rule unless you first know which side is sorted.

## Common Mistakes

The branch conditions are the whole problem. Most bugs come from discarding a side without proving whether the target could be there.

| Mistake | Why it fails |
| :--- | :--- |
| `if nums[mid] > nums[left]` | Fails to classify one-element left halves as sorted. Prefer `nums[left] <= nums[mid]`. |
| `if nums[left] < target < nums[mid]` | Excludes the left endpoint even though target could equal `nums[left]`. |
| `if nums[mid] < target < nums[right]` | Excludes the right endpoint even though target could equal `nums[right]`. |
| Forgetting the equality check first | Makes the range checks more awkward and easier to get wrong. |

## Complexity

Every iteration discards about half the current search window. The algorithm only stores indexes.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log n)` | Binary search over the rotated array. |
| Space | `O(1)` | No extra data structure is used. |

## Interview Questions and Answers

**Q: What makes this different from normal binary search?**  
A: The whole array is not sorted anymore, so you cannot blindly compare `nums[mid]` to `target`. First identify which half is sorted.

**Q: How do you know the left half is sorted?**  
A: If `nums[left] <= nums[mid]`, then the values from `left` to `mid` are in ascending order.

**Q: What do you do after finding the sorted half?**  
A: Check whether `target` falls inside that half's endpoint range. If it does, search that half; otherwise search the other half.

**Q: Why include equality in `nums[left] <= nums[mid]`?**  
A: When `left == mid`, the left half has one element and is sorted. This matters for small windows like `[3, 1]`.

**Q: Why do the range checks exclude `mid`?**  
A: `nums[mid] == target` was already checked. If mid is not the target, it can be discarded.

**Q: What changes if duplicates are allowed?**  
A: The condition `nums[left] <= nums[mid]` can stop proving which side is sorted. You may need duplicate-shrinking logic, and worst-case time can become `O(n)`.

**Q: What should you say out loud in an interview?**  
A: "At each mid, one side is sorted. I check if the target belongs in that sorted side; if yes I keep it, otherwise I discard it."

## Sources

- [LeetCode: Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)
- Problem statement and partial solution provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Find Minimum in Rotated Sorted Array](./find-minimum-in-rotated-sorted-array.md)
