---
title: "Search in a 2D Matrix"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, matrix, neetcode]
aliases: []
---

# Search in a 2D Matrix

[toc]

> **TL;DR:** Treat the matrix like one sorted 1D array without actually copying it. Binary search over indexes from `0` to `rows * cols - 1`, then convert each midpoint back into `(row, col)` with division and modulo.

## Vocabulary

**Matrix**

A rectangular 2D list. In this problem it has `rows` horizontal rows and `cols` vertical columns.

**Flattened index**

The imaginary 1D position of a matrix cell if all rows were placed end-to-end from left to right.

**Row-major order**

The order Python lists use for a 2D matrix: finish row 0, then row 1, then row 2, and so on.

**Inclusive search range**

A binary search range where both `left` and `right` are valid candidate indexes. That is why exact search uses `while left <= right`.

## Problem Pattern

The matrix has two important sorted properties. Each row is sorted, and the first number of each row is greater than the last number of the previous row.

That second rule is the unlock. It means the whole matrix is globally sorted if you read it row by row.

```text
matrix = [
  [ 1,  2,  4,  8],
  [10, 11, 12, 13],
  [14, 20, 30, 40]
]

flattened view:
index:  0  1  2  3   4   5   6   7   8   9  10  11
value:  1  2  4  8  10  11  12  13  14  20  30  40
```

We do not create this flattened array. We only pretend it exists, because the midpoint index can be mapped back into the real matrix.

> [!IMPORTANT]
> The binary search compares `value` to `target`, not `mid` to `target`. `mid` is a position. `target` is a number stored in the matrix.

## Why Store Rows and Columns

The dimensions tell us how many total searchable cells exist and how to convert a 1D index back into a 2D coordinate. The problem constraints say the matrix is non-empty, so `matrix[0]` is safe on NeetCode.

```python
rows = len(matrix)
cols = len(matrix[0])
```

`rows` is the number of lists inside `matrix`. `cols` is the number of values inside each row.

For a 3 by 4 matrix:

```text
rows = 3
cols = 4
total cells = rows * cols = 12
```

## Why Right Is Rows Times Cols Minus One

Binary search needs the first and last valid indexes of the imaginary flattened array. Since Python indexes start at `0`, an array with `rows * cols` cells ends at `rows * cols - 1`.

```python
left = 0
right = rows * cols - 1
```

For a 3 by 4 matrix, there are 12 cells:

```text
valid indexes: 0 through 11
right = 3 * 4 - 1 = 11
```

If you used `right = rows * cols`, then `right` would be `12`, which is one past the last valid index. That would eventually map to a row that does not exist.

## How Mid Maps Back to Row and Column

The midpoint is a flattened 1D index. To read the actual value, convert that index into the matrix coordinate.

```python
row = mid // cols
col = mid % cols
value = matrix[row][col]
```

Integer division asks: "How many full rows fit before this index?" That gives the row. Modulo asks: "How far into the current row am I?" That gives the column.

For `cols = 4`:

| `mid` | `row = mid // 4` | `col = mid % 4` | Matrix cell |
| ---: | ---: | ---: | :--- |
| `0` | `0` | `0` | `matrix[0][0]` |
| `3` | `0` | `3` | `matrix[0][3]` |
| `4` | `1` | `0` | `matrix[1][0]` |
| `5` | `1` | `1` | `matrix[1][1]` |
| `11` | `2` | `3` | `matrix[2][3]` |

This works because every row has exactly `cols` values. Every time the flattened index crosses another multiple of `cols`, it moves down one row.

> [!TIP]
> Python can combine the conversion into `row, col = divmod(mid, cols)`. Writing the two lines separately is often clearer while learning.

## Binary Search Logic

This is exact search. Once we inspect `mid`, either we found the target or `mid` is no longer useful.

That means we discard `mid` with `mid + 1` or `mid - 1`, and the loop must be `while left <= right` so the final remaining candidate still gets checked.

```python
while left <= right:
    mid = (left + right) // 2

    row = mid // cols
    col = mid % cols
    value = matrix[row][col]

    if value == target:
        return True
    elif value < target:
        left = mid + 1
    else:
        right = mid - 1
```

If `value < target`, everything at `mid` and before it is too small. If `value > target`, everything at `mid` and after it is too large.

## Full Python Solution

This version follows the exact binary search template from the main binary search note. The only special part is translating `mid` into `row` and `col`.

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        rows = len(matrix)
        cols = len(matrix[0])

        left = 0
        right = rows * cols - 1

        while left <= right:
            mid = (left + right) // 2

            row = mid // cols
            col = mid % cols
            value = matrix[row][col]

            if value == target:
                return True
            elif value < target:
                left = mid + 1
            else:
                right = mid - 1

        return False
```

The NeetCode constraints guarantee at least one row and one column. If this appeared outside LeetCode with possible empty input, add a guard at the top.

```python
if not matrix or not matrix[0]:
    return False
```

## Dry Run

Use the sample matrix and search for `target = 10`. The flattened index range starts at `0..11`.

```text
matrix = [
  [ 1,  2,  4,  8],
  [10, 11, 12, 13],
  [14, 20, 30, 40]
]
```

The search lands on the target after using the flattened midpoint math.

| Step | `left` | `right` | `mid` | `(row, col)` | `value` | Decision |
| ---: | ---: | ---: | ---: | :---: | ---: | :--- |
| 1 | `0` | `11` | `5` | `(1, 1)` | `11` | Too large, move `right` to `4` |
| 2 | `0` | `4` | `2` | `(0, 2)` | `4` | Too small, move `left` to `3` |
| 3 | `3` | `4` | `3` | `(0, 3)` | `8` | Too small, move `left` to `4` |
| 4 | `4` | `4` | `4` | `(1, 0)` | `10` | Found |

The last step is why `while left <= right` matters. When `left == right`, there is still one candidate left to inspect.

## Edge Cases

These are the cases that usually expose off-by-one or wrong-comparison bugs.

| Case | Why it matters |
| :--- | :--- |
| One cell matrix, target exists | `while left < right` would skip the only cell. |
| One row | The row formula still works because `row` stays `0`. |
| One column | The column formula still works because `col` is always `0`. |
| Target smaller than first value | Search eventually moves `right` below `left` and returns `False`. |
| Target larger than last value | Search eventually moves `left` past `right` and returns `False`. |
| Target at first or last cell | Confirms `right = rows * cols - 1` includes the last cell. |
| Negative values | No special handling needed; sorted order still drives the search. |

> [!WARNING]
> A common bug is `elif mid < target:`. That compares an index to the target value. Always compare `value < target`.

## Complexity

There are `rows * cols` total cells in the imaginary flattened array. Binary search cuts that search space in half each iteration.

| Resource | Complexity | Reason |
| :--- | :--- | :--- |
| Time | `O(log(rows * cols))` | Binary search over every cell position. |
| Space | `O(1)` | No flattened array is created. |

## Interview Questions and Answers

**Q: Why can we binary search the whole matrix instead of each row?**  
A: Because every row is sorted and each row starts after the previous row ends. Read row by row, the matrix behaves like one sorted array.

**Q: Why is `right = rows * cols - 1`?**  
A: `rows * cols` is the number of cells, but zero-based indexes end one before the count. The last valid flattened index is `rows * cols - 1`.

**Q: What does `row = mid // cols` mean?**  
A: It counts how many complete rows fit before index `mid`, which gives the row number.

**Q: What does `col = mid % cols` mean?**  
A: It gives the leftover offset inside the current row, which is the column number.

**Q: Why use `while left <= right`?**  
A: This is exact search and both ends are valid candidates. When only one candidate remains, `left == right`, and we still need to check it.

**Q: What should the binary search compare against the target?**  
A: Compare `matrix[row][col]`, stored as `value`, against `target`. Do not compare `mid` against `target`.

## Sources

- [NeetCode: Search a 2D Matrix](https://neetcode.io/problems/search-2d-matrix/question?list=neetcode150)
- Conversation with user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
