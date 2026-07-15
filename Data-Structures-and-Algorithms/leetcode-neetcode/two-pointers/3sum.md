---
title: "3Sum"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, two-pointers, sorting, arrays, neetcode]
aliases: []
---

# 3Sum

[toc]

> **TL;DR:** Sort the array, fix one number, then solve a two-sum problem on the remaining right side with two pointers. Skip duplicates for the fixed number and for matching pairs.

## Vocabulary

**Triplet**

Three values whose sum is zero.

**Fixed anchor**

The first value of a triplet, chosen by the outer loop.

**Duplicate skipping**

Moving past repeated values so the output does not contain the same triplet more than once.

## Underlying Concept

3Sum reduces to repeated Two Sum II after sorting. Once you fix `nums[i]`, the remaining two numbers must add to `-nums[i]`.

```text
nums[i] + nums[left] + nums[right] = 0

nums[left] + nums[right] = -nums[i]
```

Because the array is sorted, the inner two-pointer scan can move based on whether the sum is too small or too large.

> [!IMPORTANT]
> Sorting is what makes duplicate control and two-pointer movement possible.

## Algorithm Shape

First sort the array. Then walk through each possible anchor `i`.

For each anchor, place `left` right after `i` and `right` at the end.

```text
sorted nums = [-4, -1, -1, 0, 1, 2]

i anchors -1
left starts at next value
right starts at 2
```

If the three-number sum is too small, move `left` rightward. If it is too large, move `right` leftward.

## Duplicate Rules

There are two duplicate checks.

Skip repeated anchors:

```python
if i > 0 and nums[i] == nums[i - 1]:
    continue
```

After finding a valid triplet, move `left` past repeated left values so the same triplet is not emitted again.

```python
while left < right and nums[left] == nums[left - 1]:
    left += 1
```

This keeps the output unique without using a set of triplets.

## Full Python Solution

This solution returns triplets in any order, which the problem allows.

```python
from typing import List


class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        result = []

        for i in range(len(nums)):
            if i > 0 and nums[i] == nums[i - 1]:
                continue

            left = i + 1
            right = len(nums) - 1

            while left < right:
                total = nums[i] + nums[left] + nums[right]

                if total == 0:
                    result.append([nums[i], nums[left], nums[right]])
                    left += 1

                    while left < right and nums[left] == nums[left - 1]:
                        left += 1
                elif total < 0:
                    left += 1
                else:
                    right -= 1

        return result
```

The outer loop fixes one number. The inner loop finds all unique pairs to the right that complete the sum.

## Dry Run

Use `nums = [-1, 0, 1, 2, -1, -4]`. Sort first.

```text
nums = [-4, -1, -1, 0, 1, 2]
```

When `i = 1`, the anchor is `-1`.

| Step | Anchor | Left | Right | Total | Decision |
| ---: | ---: | ---: | ---: | ---: | :--- |
| 1 | `-1` | `-1` | `2` | `0` | add `[-1, -1, 2]` |
| 2 | `-1` | `0` | `2` | `1` | too large, move `right` |
| 3 | `-1` | `0` | `1` | `0` | add `[-1, 0, 1]` |

The next anchor `-1` is skipped because it is a duplicate of the previous anchor.

## Edge Cases

These cases mostly test duplicate handling.

| Case | Example | Result |
| :--- | :--- | :--- |
| All zeroes | `[0, 0, 0]` | `[[0, 0, 0]]` |
| No triplet | `[0, 1, 1]` | `[]` |
| Repeated anchors | `[-1, -1, 0, 1]` | one `[-1, 0, 1]` |
| Less than three useful values | still length at least 3 | may return `[]` |
| All positive | `[1, 2, 3]` | `[]` |

> [!WARNING]
> Do not forget to skip duplicate anchors. Without that, the same triplet can appear multiple times.

## Complexity

Sorting costs `O(n log n)`, and the nested anchor-plus-two-pointer scan costs `O(n^2)`.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(n^2)` | For each anchor, scan the rest with two pointers. |
| Space | `O(1)` extra | Ignoring output storage and sort implementation details. |

## Interview Questions and Answers

**Q: What is the underlying concept?**  
A: Sort the array, then reduce each anchor to a two-sum problem.

**Q: Why sort first?**  
A: Sorting lets the two pointers move predictably and makes duplicates adjacent.

**Q: How do you avoid duplicate triplets?**  
A: Skip duplicate anchors and skip duplicate left values after recording a triplet.

**Q: What should you say out loud in an interview?**  
A: "I sort, fix one number, then use two pointers to find pairs that sum to the negative anchor while skipping duplicates."

## Sources

- [NeetCode: 3Sum](https://neetcode.io/problems/three-integer-sum/question?list=neetcode150)
- Problem request from user on 2026-07-15.

## Related

- [Two Pointers](../../02-two-pointers.md)
- [Two Sum II Input Array Is Sorted](./two-sum-ii-input-array-is-sorted.md)
