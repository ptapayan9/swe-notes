---
title: "Sqrt(x)"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, math, neetcode]
aliases: []
---

# Sqrt(x)

[toc]

> **TL;DR:** This is binary search on the answer. Find the largest integer `mid` where `mid * mid <= x`.

## Vocabulary

**Integer square root**

The greatest integer whose square is less than or equal to `x`.

**Candidate answer**

A value that works so far but might not be the largest working value.

**Binary search on answer**

Searching over possible answers instead of searching inside an input array.

## Underlying Concept

The answer is an integer from `0` to `x`. Smaller numbers are easier to square, and larger numbers eventually become too big.

```text
x = 8

k:       0  1  2  3  4
k*k<=8:  T  T  T  F  F
                  ^
                  first too big
```

We want the last true value, which is `2`.

> [!IMPORTANT]
> Do not use floating-point square root for this problem. The binary-search version avoids rounding issues and teaches the "largest valid answer" pattern.

## Binary Search Logic

When `mid * mid <= x`, `mid` is valid. Save it, then search right for a larger valid answer.

When `mid * mid > x`, `mid` is too large. Search left.

```python
if mid * mid <= x:
    answer = mid
    left = mid + 1
else:
    right = mid - 1
```

This uses the exact-search style loop `while left <= right` because each checked `mid` is either saved as a valid candidate or discarded.

## Full Python Solution

This solution returns the floor of the real square root.

```python
class Solution:
    def mySqrt(self, x: int) -> int:
        left = 0
        right = x
        answer = 0

        while left <= right:
            mid = (left + right) // 2
            square = mid * mid

            if square <= x:
                answer = mid
                left = mid + 1
            else:
                right = mid - 1

        return answer
```

For a tiny optimization, `right` can be `x // 2 + 1` when `x >= 2`, but `right = x` is simpler and still logarithmic.

## Dry Run

Use `x = 8`.

| Step | `left` | `right` | `mid` | `mid * mid` | Decision |
| ---: | ---: | ---: | ---: | ---: | :--- |
| 1 | `0` | `8` | `4` | `16` | too large, `right = 3` |
| 2 | `0` | `3` | `1` | `1` | valid, save `1`, `left = 2` |
| 3 | `2` | `3` | `2` | `4` | valid, save `2`, `left = 3` |
| 4 | `3` | `3` | `3` | `9` | too large, `right = 2` |

The search ends with `answer = 2`.

## Edge Cases

These cases explain why `left` can start at `0`.

| Case | Result | Why |
| :--- | ---: | :--- |
| `x = 0` | `0` | Zero squared is zero. |
| `x = 1` | `1` | One squared is one. |
| Perfect square `x = 16` | `4` | Exact square root. |
| Non-perfect square `x = 8` | `2` | Return the floor. |
| Large x | valid | Multiplication is safe in Python. |

> [!WARNING]
> If coding this in a fixed-width integer language, avoid overflow by comparing `mid <= x // mid` instead of computing `mid * mid`.

## Complexity

The candidate range is cut in half each time.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log x)` | Binary search over possible square roots. |
| Space | `O(1)` | Only pointers and an answer variable are stored. |

## Interview Questions and Answers

**Q: What are we binary searching over?**  
A: Possible integer square root values from `0` to `x`.

**Q: Why save `answer` when `mid * mid <= x`?**  
A: `mid` works, but a larger value might also work.

**Q: Why return the floor?**  
A: The problem asks for the integer square root, so decimals are truncated.

**Q: What pattern is this?**  
A: Largest valid answer, where valid means `mid * mid <= x`.

**Q: What should you say out loud in an interview?**  
A: "I binary search for the largest integer whose square does not exceed x."

## Sources

- [LeetCode: Sqrt(x)](https://leetcode.com/problems/sqrtx/)
- Problem list provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Koko Eating Bananas](./koko-eating-bananas.md)
