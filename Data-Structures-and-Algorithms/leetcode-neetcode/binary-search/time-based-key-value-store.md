---
title: "Time Based Key-Value Store"
created: 2026-07-15
updated: 2026-07-15
tags: [dsa, binary-search, hash-map, design, neetcode]
aliases: []
---

# Time Based Key-Value Store

[toc]

> **TL;DR:** Store each key's history as a sorted list of `(timestamp, value)` pairs. `set` appends to that history, and `get` uses binary search to find the largest timestamp less than or equal to the requested timestamp.

## Vocabulary

**Time-based key-value store**

A map where one key can have many values over time. The value returned depends on the timestamp being queried.

**Version history**

The ordered list of values stored for one key, each paired with the timestamp when it was set.

**Floor query**

A search for the greatest value that is less than or equal to a target. Here, we want the greatest stored timestamp that is less than or equal to the query timestamp.

**Upper bound**

The first position greater than the query. If you find the upper bound, the answer is one position before it.

## Underlying Concept

This problem combines a hash map with binary search. The hash map jumps to the right key, and binary search finds the right timestamp inside that key's history.

The structure looks like this:

```text
store = {
  "alice": [(1, "happy"), (3, "sad")],
  "bob":   [(2, "tired"), (5, "ready")]
}
```

When we call `get("alice", 2)`, there is no exact timestamp `2`. The correct answer is the value at timestamp `1`, because `1` is the largest timestamp less than or equal to `2`.

> [!IMPORTANT]
> The core question is not "does this exact timestamp exist?" The core question is "what is the latest timestamp I have that does not go past the query?"

## Why a Normal Hash Map Is Not Enough

A simple dictionary can answer exact lookups, but this problem asks for the previous value when there is no exact timestamp.

```text
set("alice", "happy", 1)
get("alice", 2) -> "happy"
```

If we only stored this:

```python
store["alice"][1] = "happy"
```

then timestamp `2` would not be found directly. We need an ordered history so we can step backward to the closest valid timestamp.

## Data Structure Shape

Use a dictionary where each key maps to a list of timestamp-value pairs.

```python
self.store = {
    key: [(timestamp1, value1), (timestamp2, value2)]
}
```

The problem says `set` timestamps are strictly increasing. That means each key's list stays sorted as we append new pairs.

```python
self.store[key].append((timestamp, value))
```

No sorting is needed after each insert. That is the detail that makes `set` fast and makes binary search possible during `get`.

## How Get Works

For `get(key, timestamp)`, first get that key's history. If the key does not exist, return an empty string.

Then binary search for the largest stored timestamp less than or equal to the query timestamp.

```text
history for "alice":

index:      0          1
pair:    (1,happy)  (3,sad)

get("alice", 2)

valid timestamps <= 2: [1]
largest valid timestamp: 1
answer: "happy"
```

The binary search keeps a candidate answer. Every time `history[mid].timestamp <= query`, that value is valid, but there may be a later valid timestamp to the right.

## Manual Binary Search

This is a "largest valid" binary search. The valid condition is `stored_timestamp <= query_timestamp`.

```python
left = 0
right = len(history) - 1
answer = ""

while left <= right:
    mid = (left + right) // 2
    stored_timestamp, value = history[mid]

    if stored_timestamp <= timestamp:
        answer = value
        left = mid + 1
    else:
        right = mid - 1

return answer
```

If `stored_timestamp <= timestamp`, the current value is legal, so save it. Then search right to see if there is an even newer legal timestamp.

If `stored_timestamp > timestamp`, that timestamp is too far in the future, so search left.

## Full Python Solution

This version uses a regular dictionary and creates a new list the first time a key appears. The `get` method does all the interesting work.

```python
class TimeMap:

    def __init__(self):
        self.store = {}

    def set(self, key: str, value: str, timestamp: int) -> None:
        if key not in self.store:
            self.store[key] = []

        self.store[key].append((timestamp, value))

    def get(self, key: str, timestamp: int) -> str:
        if key not in self.store:
            return ""

        history = self.store[key]
        left = 0
        right = len(history) - 1
        answer = ""

        while left <= right:
            mid = (left + right) // 2
            stored_timestamp, value = history[mid]

            if stored_timestamp <= timestamp:
                answer = value
                left = mid + 1
            else:
                right = mid - 1

        return answer
```

This pattern is easier to understand than a one-line `bisect` solution because the candidate answer is visible.

## Dry Run

Use the sample operations for `"alice"`.

```text
set("alice", "happy", 1)
get("alice", 1)
get("alice", 2)
set("alice", "sad", 3)
get("alice", 3)
```

After both `set` calls, the stored history is:

```text
alice -> [(1, "happy"), (3, "sad")]
```

The query `get("alice", 2)` searches for the largest timestamp less than or equal to `2`.

| Step | `left` | `right` | `mid` | Stored pair | Decision |
| ---: | ---: | ---: | ---: | :--- | :--- |
| 1 | `0` | `1` | `0` | `(1, "happy")` | `1 <= 2`, save `"happy"`, search right |
| 2 | `1` | `1` | `1` | `(3, "sad")` | `3 > 2`, search left |

The search ends with `answer = "happy"`.

## Upper Bound Mental Model

Another way to think about `get` is this: find the first timestamp greater than the query timestamp, then step one position left.

```text
timestamps: [1, 3, 7, 10]
query: 6

first timestamp > 6 is 7
step left to 3
answer is the value stored at timestamp 3
```

That is called an upper-bound search. The manual code above achieves the same result by saving the best valid value as it searches.

## Edge Cases

These cases usually expose wrong binary search boundaries.

| Case | Example | Expected result |
| :--- | :--- | :--- |
| Key does not exist | `get("bob", 1)` | `""` |
| Query before first timestamp | history `[(5, "x")]`, query `3` | `""` |
| Query equals exact timestamp | history `[(1, "a"), (3, "b")]`, query `3` | `"b"` |
| Query between timestamps | history `[(1, "a"), (3, "b")]`, query `2` | `"a"` |
| Query after latest timestamp | history `[(1, "a"), (3, "b")]`, query `10` | `"b"` |
| Multiple keys | `alice` and `bob` histories | Search only that key's list |

> [!WARNING]
> Do not return `""` just because the exact timestamp is missing. The problem asks for the latest previous timestamp, not exact-match-only lookup.

## Complexity

Let `m` be the number of values stored for the specific key being queried.

| Operation | Time | Space | Reason |
| :--- | :--- | :--- | :--- |
| `set` | `O(1)` | `O(1)` extra | Append to that key's list. |
| `get` | `O(log m)` | `O(1)` extra | Binary search within one key's history. |
| Total storage | `O(n)` | `O(n)` | Store every timestamp-value pair. |

## Interview Questions and Answers

**Q: What is the underlying concept?**  
A: A hash map from key to sorted version history, plus binary search for a floor timestamp.

**Q: Why do we store a list for each key instead of one value?**  
A: The same key can have multiple values at different timestamps, and queries may ask for a past version.

**Q: Why is each list sorted?**  
A: The problem guarantees `set` timestamps are strictly increasing, so appending keeps the history ordered by timestamp.

**Q: What does `get` search for?**  
A: The largest stored timestamp less than or equal to the query timestamp.

**Q: Why save `answer` and then move right?**  
A: The current timestamp is valid, but a later timestamp might also be valid and would be a better answer.

**Q: What should you say out loud in an interview?**  
A: "I store each key's values as a sorted timestamp history. For `get`, I binary search that key's history for the rightmost timestamp that is at most the query timestamp."

## Sources

- [LeetCode: Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/)
- Problem statement provided by user on 2026-07-15.

## Related

- [Binary Search](../../01-binary-search.md)
- [Koko Eating Bananas](./koko-eating-bananas.md)
