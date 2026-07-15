---
title: "Two Sum II Input Array Is Sorted"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, two-pointers, sorted-array, neetcode]
aliases: []
---

# Two Sum II Input Array Is Sorted

[toc]

> **TL;DR:** Because the array is sorted, use one pointer at the smallest value and one at the largest value. If the sum is too small, move left rightward; if the sum is too large, move right leftward.

## Vocabulary

**Sorted array**

An array where values appear in non-decreasing order.

**Opposite-end pointers**

One pointer starts at the first element and the other starts at the last element.

**One-indexed answer**

The returned positions are `left + 1` and `right + 1`, not normal Python zero-based indexes.

## Underlying Concept

This problem is the classic sorted two-sum pattern. The sorted order tells us how to move a pointer when the current sum is wrong.

```text
numbers = [1, 2, 3, 4]
target = 3

left points to 1
right points to 4
sum = 5, too large, so move right leftward
```

The left pointer controls the smaller value. The right pointer controls the larger value.

> [!IMPORTANT]
> The sorted array is the reason this can be `O(n)` with `O(1)` extra space. Without sorting, this pointer movement would not be justified.

## Pointer Logic

At each step, compare the current sum with the target.

```python
current = numbers[left] + numbers[right]
```

If `current` is too small, the left value is too small to work with this right value, so move `left`.

If `current` is too large, the right value is too large to work with this left value, so move `right`.

| Condition | Meaning | Move |
| :--- | :--- | :--- |
| `current == target` | Found the pair. | return indexes |
| `current < target` | Need a larger sum. | `left += 1` |
| `current > target` | Need a smaller sum. | `right -= 1` |

## Full Python Solution

The problem guarantees exactly one valid solution. That lets the function return as soon as it finds the pair.

```python
from typing import List


class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        left = 0
        right = len(numbers) - 1

        while left < right:
            current = numbers[left] + numbers[right]

            if current == target:
                return [left + 1, right + 1]
            elif current < target:
                left += 1
            else:
                right -= 1

        return []
```

The returned indexes add `1` because the problem asks for one-indexed positions.

## Dry Run

Use `numbers = [1, 2, 3, 4]` and `target = 3`.

| Step | `left` | `right` | Values | Sum | Decision |
| ---: | ---: | ---: | :--- | ---: | :--- |
| 1 | `0` | `3` | `1 + 4` | `5` | too large, move `right` |
| 2 | `0` | `2` | `1 + 3` | `4` | too large, move `right` |
| 3 | `0` | `1` | `1 + 2` | `3` | found |

Return `[1, 2]`, because the answer is one-indexed.

## Edge Cases

These cases make the pointer movement and indexing clear.

| Case | Example | Result |
| :--- | :--- | :--- |
| Pair at the start | `[1, 2, 7]`, target `3` | `[1, 2]` |
| Pair at the ends | `[1, 2, 4, 8]`, target `9` | `[1, 4]` |
| Negative values | `[-3, -1, 2, 4]`, target `1` | `[1, 4]` |
| Duplicate values | `[1, 2, 2, 4]`, target `4` | `[2, 3]` |

> [!WARNING]
> Do not forget the one-indexed output. Returning `[left, right]` is off by one.

## Complexity

Each pointer moves inward at most `n` times total.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(n)` | One linear pass with two pointers. |
| Space | `O(1)` | No hash map is used. |

## Interview Questions and Answers

**Q: Why does moving the left pointer increase the sum?**  
A: The array is sorted, so moving left rightward chooses a value that is greater than or equal to the current one.

**Q: Why does moving the right pointer decrease the sum?**  
A: Moving right leftward chooses a value that is less than or equal to the current one.

**Q: Why not use a hash map?**  
A: The problem requires constant extra space and the sorted order gives a cleaner two-pointer solution.

**Q: What should you say out loud in an interview?**  
A: "I compare the smallest and largest remaining values. If the sum is too small I raise the low side; if too large I lower the high side."

## Sources

- [NeetCode: Two Sum II Input Array Is Sorted](https://neetcode.io/problems/two-integer-sum-ii/question?list=neetcode150)
- Problem request from user on 2026-07-15.

## Related

- [Two Pointers](../../02-two-pointers.md)
- [3Sum](./3sum.md)
