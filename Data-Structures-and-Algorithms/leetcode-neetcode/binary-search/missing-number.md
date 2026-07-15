---
title: "Missing Number"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, arrays, binary-search, math, neetcode]
aliases: []
---

# Missing Number

[toc]

> **TL;DR:** The original LeetCode problem does not give a sorted array, so the best direct solution is sum or XOR in linear time. If the array is sorted, or if you sort it first for practice, binary search finds the first index where `nums[i] != i`.

## Vocabulary

**Range from zero to n**

The input has `n` numbers chosen from `0` through `n`, so exactly one number is missing.

**Index-value invariant**

In a perfectly sorted no-missing prefix, the value equals the index: `nums[i] == i`.

**First mismatch**

The first index where `nums[i] != i`. In the sorted version, this index is the missing number.

**Arithmetic sum**

The expected total of all numbers from `0` through `n`. Subtracting the actual sum reveals the missing value.

## Underlying Concept

This problem is not naturally binary search unless the numbers are sorted. The original input can arrive in any order.

For the normal LeetCode problem, use the sum formula:

```text
missing = expected sum from 0..n - actual sum of nums
```

For binary-search practice, sort the array first and look for the first broken index-value match.

```text
sorted nums = [0, 1, 3]

index:        0  1  2
value:        0  1  3

first mismatch is index 2
missing number is 2
```

> [!IMPORTANT]
> Binary search only applies after the values are sorted. Sorting makes the binary-search version slower than the direct sum or XOR solution, but it teaches a useful "first mismatch" pattern.

## Optimal Sum Solution

This is the version to use for the actual LeetCode problem. It is short, linear, and does not need sorting.

```python
from typing import List


class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        n = len(nums)
        expected = n * (n + 1) // 2
        actual = sum(nums)

        return expected - actual
```

The formula works because the complete range has exactly one more number than the input. Whatever amount is missing from the actual sum is the missing value.

## Binary Search Version

If the array is sorted, every value before the missing number equals its index. Every value after the missing number is shifted one step too high.

```python
from typing import List


class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        nums = sorted(nums)

        left = 0
        right = len(nums)

        while left < right:
            mid = (left + right) // 2

            if nums[mid] > mid:
                right = mid
            else:
                left = mid + 1

        return left
```

Use `right = len(nums)` because the answer can be `n`. For example, if `nums = [0, 1, 2]`, no index mismatches, so the missing number is `3`.

## Why the Binary Search Moves Work

After sorting, compare `nums[mid]` to `mid`.

| Check | Meaning | Move |
| :--- | :--- | :--- |
| `nums[mid] == mid` | Nothing is missing up through `mid`. | `left = mid + 1` |
| `nums[mid] > mid` | A number is missing at `mid` or earlier. | `right = mid` |

The condition `nums[mid] > mid` means the current value is too large for its index. That only happens after the missing value has shifted the array.

## Dry Run

Use `nums = [3, 0, 1]`. First sort it for the binary-search version.

```text
sorted nums = [0, 1, 3]
```

Now search for the first mismatch.

| Step | `left` | `right` | `mid` | `nums[mid]` | Decision |
| ---: | ---: | ---: | ---: | ---: | :--- |
| 1 | `0` | `3` | `1` | `1` | matches index, move `left = 2` |
| 2 | `2` | `3` | `2` | `3` | too high, move `right = 2` |

Now `left == right == 2`, so return `2`.

## Edge Cases

These cases explain why the bounds matter.

| Case | Example | Result |
| :--- | :--- | ---: |
| Missing zero | `[1, 2, 3]` | `0` |
| Missing n | `[0, 1, 2]` | `3` |
| One element, missing zero | `[1]` | `0` |
| One element, missing one | `[0]` | `1` |
| Unsorted input | `[3, 0, 1]` | Sum solution works directly |

> [!WARNING]
> Do not use binary search on the unsorted input. Binary search needs a sorted invariant, and the original problem does not provide one.

## Complexity

The direct sum solution is better for this problem. The binary-search version is useful only if the array is already sorted or if you are practicing the pattern.

| Solution | Time | Space | Reason |
| :--- | :--- | :--- | :--- |
| Sum | `O(n)` | `O(1)` | One pass and a formula. |
| Binary search after sorting | `O(n log n)` | depends on sort | Sorting dominates. |
| Binary search if already sorted | `O(log n)` | `O(1)` | Search for first mismatch. |

## Interview Questions and Answers

**Q: What is the best solution for the original problem?**  
A: Use the expected sum from `0` through `n` minus the actual sum.

**Q: Why is the binary-search version not the best default?**  
A: The input is not guaranteed sorted, and sorting costs `O(n log n)`.

**Q: When does binary search make sense?**  
A: When the array is already sorted or when the interview specifically asks for the first-mismatch binary search pattern.

**Q: Why can the answer be `len(nums)`?**  
A: If every index matches its value, then the missing value is the final number `n`.

**Q: What should you say out loud in an interview?**  
A: "For the actual unsorted problem, I use the sum formula. If sorted, I can binary search for the first index where value is greater than index."

## Sources

- [LeetCode: Missing Number](https://leetcode.com/problems/missing-number/)
- Problem list provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Search Insert Position](./search-insert-position.md)
