---
title: "Median of Two Sorted Arrays"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, partition, arrays, neetcode]
aliases: []
---

# Median of Two Sorted Arrays

[toc]

> **TL;DR:** Do not merge the arrays. Binary search how many elements to take from the smaller array's left side so the combined left partition and right partition are correctly ordered.

## Vocabulary

**Median**

The middle value after all numbers are sorted. If the total count is even, the median is the average of the two middle values.

**Partition**

A split between elements. For an array of length `m`, there are `m + 1` possible partitions: before the first element, between elements, and after the last element.

**Left partition**

The group of elements that should be on the left side of the combined sorted order.

**Right partition**

The group of elements that should be on the right side of the combined sorted order.

**Sentinel**

A fake boundary value such as negative infinity or positive infinity used when a partition sits before the first element or after the last element.

## Underlying Concept

This problem is binary search over a partition, not binary search for a target value. We are trying to split both arrays so that everything on the left side is less than or equal to everything on the right side.

For the merged sorted array, the median lives at the boundary between the left and right halves.

```text
combined sorted view:

left half       right half
[1, 2]       |  [3, 4]
             ^
             median boundary
```

Instead of building that combined array, we find the same boundary using the two already-sorted arrays.

> [!IMPORTANT]
> The question is not "where is the median value?" The question is "where can I cut both arrays so the left side contains the correct number of values and every left value is <= every right value?"

## The Partition Picture

Suppose:

```text
A = [1, 3]
B = [2, 4]
```

One valid partition is:

```text
A: [1] | [3]
B: [2] | [4]

left side:  [1, 2]
right side: [3, 4]
```

This partition is valid because the largest value on the left is `2`, and the smallest value on the right is `3`. For an even total count, the median is their average.

```text
(2 + 3) / 2 = 2.5
```

## What We Binary Search

Always binary search the smaller array. Call it `A`, and call the other array `B`.

```python
if len(nums1) > len(nums2):
    nums1, nums2 = nums2, nums1
```

If `A` has length `m`, there are `m + 1` possible cut positions. The cut position `i` means "put `i` elements from `A` on the left side."

```text
A = [1, 3, 8]

i = 0: []        | [1, 3, 8]
i = 1: [1]       | [3, 8]
i = 2: [1, 3]    | [8]
i = 3: [1, 3, 8] | []
```

Once we pick `i`, the cut in `B` is forced. The combined left side must contain half of the total elements.

```python
total = len(A) + len(B)
half = (total + 1) // 2

j = half - i
```

`j` means "put `j` elements from `B` on the left side."

## The Four Boundary Values

For any pair of cuts `i` and `j`, only four values matter:

```text
A_left   A_right
B_left   B_right
```

They are the values immediately around the two cuts.

```python
A_left = A[i - 1] if i > 0 else float("-inf")
A_right = A[i] if i < len(A) else float("inf")

B_left = B[j - 1] if j > 0 else float("-inf")
B_right = B[j] if j < len(B) else float("inf")
```

The sentinels handle cuts at the edges. If the cut takes nothing from `A`, there is no real `A_left`, so negative infinity is safe. If the cut takes everything from `A`, there is no real `A_right`, so positive infinity is safe.

## The Valid Partition Rule

A partition is valid when the largest value on the left side is less than or equal to the smallest value on the right side.

Since `A` and `B` are individually sorted, we only need two cross-checks:

```python
A_left <= B_right and B_left <= A_right
```

If both are true, the split is correct.

```text
A: [1] | [3]
B: [2] | [4]

A_left = 1, A_right = 3
B_left = 2, B_right = 4

1 <= 4 is true
2 <= 3 is true

valid partition
```

> [!TIP]
> The sorted-array property is why only four boundary values matter. You do not need to compare every value on the left to every value on the right.

## How To Move The Search

If the partition is invalid, one side has crossed too far.

```python
if A_left > B_right:
    right = i - 1
else:
    left = i + 1
```

If `A_left > B_right`, we took too many elements from `A`. Move the cut in `A` left.

If `B_left > A_right`, we did not take enough elements from `A`. Move the cut in `A` right.

```text
Problem: A_left is too big

A: [1, 8] | [9]
B: [2]    | [3, 4]

8 > 3, so A contributed too many left-side elements.
Move A's cut left.
```

```text
Problem: B_left is too big

A: []        | [8, 9]
B: [1, 2, 3] | [4]

3 <= 8 is true, but if B_left were bigger than A_right,
A would need to contribute more to the left side.
Move A's cut right.
```

## Full Python Solution

This solution uses `while left <= right` because we are searching over cut positions in `A`, including the edges `0` and `len(A)`.

```python
from typing import List


class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        A = nums1
        B = nums2

        if len(A) > len(B):
            A, B = B, A

        total = len(A) + len(B)
        half = (total + 1) // 2

        left = 0
        right = len(A)

        while left <= right:
            i = (left + right) // 2
            j = half - i

            A_left = A[i - 1] if i > 0 else float("-inf")
            A_right = A[i] if i < len(A) else float("inf")

            B_left = B[j - 1] if j > 0 else float("-inf")
            B_right = B[j] if j < len(B) else float("inf")

            if A_left <= B_right and B_left <= A_right:
                if total % 2 == 1:
                    return float(max(A_left, B_left))

                return (max(A_left, B_left) + min(A_right, B_right)) / 2

            if A_left > B_right:
                right = i - 1
            else:
                left = i + 1
```

The binary search is over the smaller array, so the runtime is `O(log(min(m, n)))`, which is even tighter than the required `O(log(m + n))`.

## Dry Run: Odd Total

Use `nums1 = [1, 2]` and `nums2 = [3]`. Since `nums2` is smaller, make it `A`.

```text
A = [3]
B = [1, 2]

total = 3
half = 2
```

Try the possible cuts in `A`.

| Step | `i` | `j` | `A_left` | `A_right` | `B_left` | `B_right` | Decision |
| ---: | ---: | ---: | :--- | :--- | :--- | :--- | :--- |
| 1 | `0` | `2` | `-inf` | `3` | `2` | `inf` | Valid |

The total count is odd, so the median is the largest value on the left side.

```text
max(A_left, B_left) = max(-inf, 2) = 2
```

## Dry Run: Even Total

Use `nums1 = [1, 3]` and `nums2 = [2, 4]`.

```text
A = [1, 3]
B = [2, 4]

total = 4
half = 2
```

The valid partition is:

```text
A: [1] | [3]
B: [2] | [4]
```

The left side has `[1, 2]`, and the right side has `[3, 4]`.

```text
left max = 2
right min = 3
median = (2 + 3) / 2 = 2.5
```

## Edge Cases

These cases are why the sentinel values and smaller-array swap matter.

| Case | Example | Why it matters |
| :--- | :--- | :--- |
| One array empty | `[]`, `[1]` | Cut can sit at the edge of the empty array. |
| Odd total length | `[1, 2]`, `[3]` | Return the max of the left partition. |
| Even total length | `[1, 3]`, `[2, 4]` | Average the two middle boundary values. |
| All values from one array are smaller | `[1, 2]`, `[3, 4]` | Valid cut may be at an array edge. |
| Duplicates | `[1, 2, 2]`, `[2, 3]` | Use `<=`, not `<`, in the valid partition rule. |
| Different lengths | `[1]`, `[2, 3, 4, 5, 6]` | Binary search the smaller array to keep cuts safe. |
| Both arrays empty | `[]`, `[]` | Not a valid LeetCode input; no median exists. |

> [!WARNING]
> Do not binary search values directly. The key move is binary searching how many elements from the smaller array belong on the left side.

## Common Mistakes

Most bugs come from treating `i` like an index instead of a count. `i = 0` means take no elements from `A`, and `i = len(A)` means take all elements from `A`.

| Mistake | Why it fails |
| :--- | :--- |
| Binary searching the larger array | Can make `j` fall outside `B` more easily and complicates edge cases. |
| Using `half = total // 2` | For odd totals, the left side should hold one extra element. |
| Forgetting sentinels | Edge cuts need safe fake values. |
| Using `<` instead of `<=` | Duplicate boundary values should still be valid. |
| Returning `min(A_right, B_right)` for odd totals | With the chosen `half`, odd medians live at `max(A_left, B_left)`. |

## Complexity

Let `m` and `n` be the lengths of the two input arrays.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log(min(m, n)))` | Binary search only the smaller array's cut positions. |
| Space | `O(1)` | No merged array is built. |

## Interview Questions and Answers

**Q: What is the underlying concept?**  
A: Binary search for a valid partition between the left and right halves of the combined sorted order.

**Q: Why not merge the arrays?**  
A: Merging takes linear time. The problem requires logarithmic time.

**Q: Why binary search the smaller array?**  
A: It keeps the search fast and keeps the forced cut in the other array within valid bounds.

**Q: What does `i` mean?**  
A: `i` is the number of elements taken from `A` into the left partition. It is a cut position, not necessarily an element index.

**Q: What does `j = half - i` mean?**  
A: Once `i` elements come from `A`, the remaining left-side elements must come from `B`.

**Q: How do you know the partition is valid?**  
A: Check `A_left <= B_right` and `B_left <= A_right`.

**Q: What should you say out loud in an interview?**  
A: "I binary search the cut in the smaller array. The cut in the larger array is forced. When both cross-boundary checks pass, the median is built from the partition boundary values."

## Sources

- [LeetCode: Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)
- [Wikipedia: Median](https://en.wikipedia.org/wiki/Median)
- Problem statement provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Time Based Key-Value Store](./time-based-key-value-store.md)
