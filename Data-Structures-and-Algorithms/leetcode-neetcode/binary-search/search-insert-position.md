---
title: "Search Insert Position"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, lower-bound, arrays, neetcode]
aliases: []
---

# Search Insert Position

[toc]

> **TL;DR:** Find the first index where `nums[i] >= target`. That index is either the existing target position or the place where the target should be inserted.

## Vocabulary

**Insert position**

The index where `target` belongs so the array stays sorted.

**Lower bound**

The first index whose value is greater than or equal to the target.

**Candidate-preserving search**

A binary search where `mid` may be the answer, so the right side moves to `mid` instead of `mid - 1`.

## Underlying Concept

This problem is a lower-bound search. You are not only asking whether the target exists.

You are asking:

```text
Where is the first value that is not less than target?
```

Example:

```text
nums = [1, 3, 5, 6]
target = 5

first value >= 5 is at index 2
answer = 2
```

If the target is missing:

```text
nums = [1, 3, 5, 6]
target = 2

first value >= 2 is 3 at index 1
answer = 1
```

## Why Right Starts at Len Nums

The answer can be one past the last index. That happens when the target is larger than every number.

```text
nums = [1, 3, 5, 6]
target = 7

insert after the last element
answer = 4
```

So use a half-open search range:

```python
left = 0
right = len(nums)
```

Here, `right` is not a valid index. It is the possible insertion position after the array.

## Binary Search Logic

If `nums[mid] < target`, then `mid` and everything before it are too small. The answer must be after `mid`.

If `nums[mid] >= target`, then `mid` could be the first valid position, so keep it.

```python
if nums[mid] < target:
    left = mid + 1
else:
    right = mid
```

When the loop ends, `left` is the first index where the value is greater than or equal to the target.

## Full Python Solution

This is the clean lower-bound template.

```python
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums)

        while left < right:
            mid = (left + right) // 2

            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid

        return left
```

The loop uses `while left < right` because the answer remains inside the half-open range `[left, right]`.

## Dry Run

Use `nums = [1, 3, 5, 6]` and `target = 2`.

| Step | `left` | `right` | `mid` | `nums[mid]` | Decision |
| ---: | ---: | ---: | ---: | ---: | :--- |
| 1 | `0` | `4` | `2` | `5` | `5 >= 2`, keep mid, `right = 2` |
| 2 | `0` | `2` | `1` | `3` | `3 >= 2`, keep mid, `right = 1` |
| 3 | `0` | `1` | `0` | `1` | `1 < 2`, move `left = 1` |

Now `left == right == 1`, so return `1`.

## Edge Cases

These cases are why this is a lower-bound problem.

| Case | Example | Result |
| :--- | :--- | ---: |
| Target exists | `[1, 3, 5, 6]`, target `5` | `2` |
| Insert in middle | `[1, 3, 5, 6]`, target `2` | `1` |
| Insert at front | `[1, 3, 5, 6]`, target `0` | `0` |
| Insert at end | `[1, 3, 5, 6]`, target `7` | `4` |
| One element, before | `[3]`, target `1` | `0` |
| One element, after | `[3]`, target `5` | `1` |

> [!WARNING]
> Do not use `right = mid - 1` in the `nums[mid] >= target` branch. `mid` might be the first valid insertion position, so keep it.

## Complexity

Each check removes half the remaining insertion positions.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log n)` | Binary search over indexes and one after-last position. |
| Space | `O(1)` | Only pointers are stored. |

## Interview Questions and Answers

**Q: What are we searching for?**  
A: The first index where `nums[i] >= target`.

**Q: Why is this called lower bound?**  
A: It returns the lowest index that can hold the target while preserving sorted order.

**Q: Why can the answer be `len(nums)`?**  
A: If target is larger than every value, it should be inserted after the last element.

**Q: Why use `right = mid`?**  
A: `mid` may already be the first valid insertion position.

**Q: What should you say out loud in an interview?**  
A: "I am finding the first number that is not less than target. If mid is too small I go right; otherwise I keep mid and search left."

## Sources

- [LeetCode: Search Insert Position](https://leetcode.com/problems/search-insert-position/)
- Problem list provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Missing Number](./missing-number.md)
