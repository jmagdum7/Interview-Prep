# Patterns.md

> The goal is pattern recognition, not problem memorization.
> Read this until applying these feels automatic.

---

## How to use this doc

1. Read the pattern — understand the *why*, not just the code
2. Look at the signal words — train yourself to spot them in problem statements
3. Re-read after every session until the template code feels like muscle memory

---

## Table of Contents

- [Arrays & Hashing](#arrays--hashing)
  - [Hash Set — existence check](#1-hash-set--existence-check)
  - [Hash Map — frequency count](#2-hash-map--frequency-count)
  - [Hash Map — complement trick](#3-hash-map--complement-trick)
  - [Prefix Sums](#4-prefix-sums)
- [Two Pointers](#two-pointers) ← coming soon
- [Sliding Window](#sliding-window) ← coming soon
- [Stack](#stack) ← coming soon

---

---

# Arrays & Hashing

**Core idea:** Trade space for time. Store things you'd otherwise re-scan.

**The question to ask first:**
> *"What am I looking up repeatedly? Store that."*

**Signal words:** duplicate, two sum, frequency, group by, subarray sum, consecutive

---

## 1. Hash Set — existence check

**Use when:** you only need to know *if* something exists.

```python
seen = set()
for x in arr:
    if x in seen:      # O(1) — no inner loop needed
        return True
    seen.add(x)
return False
```

**Example — `[1, 2, 3, 1]`:**
```
x=1 → not seen → store
x=2 → not seen → store
x=3 → not seen → store
x=1 → SEEN → return True ✓
```

**Complexity:** O(n) time, O(n) space

---

## 2. Hash Map — frequency count

**Use when:** you need to count how many times each element appears.

```python
freq = {}
for x in arr:
    freq[x] = freq.get(x, 0) + 1
```

**Example — checking if two strings are anagrams:**
```
s="anagram" → {'a':3,'n':1,'g':1,'r':1,'m':1}
t="nagaram" → {'n':1,'a':3,'g':1,'r':1,'m':1}
Equal → True ✓
```

**Complexity:** O(n) time, O(n) space

---

## 3. Hash Map — complement trick

**Use when:** finding two elements that together satisfy a condition.

```python
seen = {}                            # value → index
for i, x in enumerate(arr):
    complement = target - x          # what we need
    if complement in seen:
        return [seen[complement], i]
    seen[x] = i
```

**Example — `[2,7,11,15]`, target=9:**
```
x=2 → need 7 → not seen → store {2:0}
x=7 → need 2 → 2 IS seen → return [0,1] ✓
```

**Complexity:** O(n) time, O(n) space

---

## 4. Prefix Sums

**Use when:** repeated questions about sums over subarrays or ranges.

```python
prefix = [0] * (len(arr) + 1)
for i in range(len(arr)):
    prefix[i+1] = prefix[i] + arr[i]

# sum from index i to j (inclusive):
range_sum = prefix[j+1] - prefix[i]
```

**Example — `[1,2,3,4]`, sum of index 1 to 2:**
```
prefix = [0, 1, 3, 6, 10]
prefix[3] - prefix[1] = 6 - 1 = 5  (2+3=5) ✓
```

**Complexity:** O(n) build, O(1) per query, O(n) space

---

## Signal words → pattern

| Problem says... | Pattern |
|---|---|
| duplicate exists | Hash set |
| two numbers sum to target | Complement map |
| frequency / count of elements | Frequency map |
| group by shared property | Map of lists |
| subarray sum equals k | Prefix sum + map |
| longest consecutive sequence | Hash set + expand |

---

---

# Two Pointers

*Coming after Arrays & Hashing is complete.*

---

# Sliding Window

*Coming after Two Pointers is complete.*

---

# Stack

*Coming after Sliding Window is complete.*
