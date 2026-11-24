# Patterns.md

> The goal is pattern recognition, not problem memorization.
> Read this until applying these feels automatic.

---

## How to use this doc

1. Read the mental trigger — understand the *why*, not just the code
2. Study the interview examples — train yourself to decode ambiguous problem statements
3. Trace through every example by hand — visualization only comes from doing, not reading
4. Re-read after every session until recognition is automatic

---

## Table of Contents

- [Part A — Blob to Pattern](#part-a--blob-to-pattern)
1. [Arrays & Hashing](#1-arrays--hashing)
2. [Two Pointers](#2-two-pointers)
3. [Sliding Window](#3-sliding-window)
4. [Stack](#4-stack)
5. [Binary Search](#5-binary-search)
6. [Linked List](#6-linked-list)
7. [Trees](#7-trees)
8. [Heap / Priority Queue](#8-heap--priority-queue)
9. [Backtracking](#9-backtracking)
10. [Graphs](#10-graphs)
11. [Dynamic Programming](#11-dynamic-programming)
12. [Greedy](#12-greedy)
13. [Intervals](#13-intervals)
14. [Bit Manipulation](#14-bit-manipulation)
- [Python Mechanics](#python-mechanics)
- [Space & Time Tradeoffs](#space--time-tradeoffs)
- [Complexity Cheat Sheet](#complexity-cheat-sheet)

---
---

## Part A — Blob to Pattern

**Step 1 — Strip the problem:**
```
1. What is the actual data?
2. What are you returning?
```

**Step 2 — Ask in order. Stop at first yes:**

    Q1. All combinations, permutations, or paths?
        → Backtracking

    Q2. Answer to bigger input depends on smaller input?
        → DP

    Q3. Elements connect to each other?
        → chain       → Linked List
        → hierarchy   → Tree
        → network     → Graph

    Q4. Each item is a range [start, end]?
        → Intervals

    Q5. Order or position matters?
        → sorted + find target or boundary     → Binary Search
        → sorted + two elements satisfy cond   → Two Pointers
        → subarray or substring                → Sliding Window
        → nesting, undo, next greater          → Stack
        → locally best choice, never undo      → Greedy

    Q6. Min or max repeatedly from changing collection?
        → Heap

    Q7. Repeatedly looking something up?
        → Arrays & Hashing

    Q8. Numbers as bits?
        → Bit Manipulation

---
---

# 1. Arrays & Hashing

### Mental Trigger
> *"What am I repeatedly looking up? Store it so each lookup is O(1) instead of O(n)."*

**Why this works:** scanning an array for something is O(n). Looking it up in a hash map or set is O(1). Every A&H problem trades O(n) space to eliminate a repeated scan.

---

### Translation Table

| You hear this... | Key | Value | Structure | Template |
|---|---|---|---|---|
| "duplicate", "seen before" | element | — | set | 1 |
| "two things sum to target" | element | index | dict | 2 |
| "count", "frequency" | element | count | dict | 3 |
| "group by property", "anagram" | signature | list | dict of lists | 3 |
| "subarray sum", "range sum" | index | running total | array | 4 |
| "top K frequent" | element | count | dict + bucket | 5 |

---

### Questions to Ask Before Coding

**Q1 — Do I need existence only, or do I need to store something with each element?**
- Existence only → `set` → Template 1
- Need index → `dict` → Template 2
- Need count or group → `dict` → Template 3

**Q2 — What is the key? What is the value?**
- This fills your `seen[key] = value` directly
- element → index → Template 2
- element → count → Template 3
- sorted signature → list of words → Template 3
- index → running total → Template 4

**Q3 — One pass or two?**
- Check and store simultaneously → one pass → Template 2
- Build full map first, then query → two pass → Templates 3, 4, 5

---

### Template 1 — Hash Set

```python
seen = set()

for x in arr:
    if x in seen:      # element already exists — act on it
        ...
    seen.add(x)        # first time — store for future lookups

return ...
```

**Example — does any value appear more than once?**
```
Q1: existence only → set
Q2: key = element, no value needed
Q3: one pass — check and store simultaneously

[1,2,3,1]:
1 → not seen → store
2 → not seen → store
3 → not seen → store
1 → IN seen → return True ✓
```
**Complexity:** O(n) time. O(n) space.

---

### Template 2 — Complement Map

```python
seen = {}                      # key: element, value: index

for i, x in enumerate(arr):
    complement = target - x    # what I need to complete the pair
    if complement in seen:
        return [seen[complement], i]
    seen[x] = i                # store for future lookups
```

**Example — which two indices sum to target=9?**
```
Q1: need index alongside element → dict
Q2: key = element, value = index
Q3: one pass — check complement, then store

[2,7,11,15], target=9:
x=2 → need 7 → not seen → store {2:0}
x=7 → need 2 → IN seen → return [0,1] ✓
```
**Complexity:** O(n) time. O(n) space.

---

### Template 3 — Frequency Map

```python
freq = {}

for x in arr:
    freq[x] = freq.get(x, 0) + 1  # get current count (default 0), increment
```

**Example — are these two strings anagrams?**
```
Q1: need counts → dict
Q2: key = character, value = count
Q3: two pass — build freq for each, then compare

"anagram" → {'a':3,'n':1,'g':1,'r':1,'m':1}
"nagaram" → {'n':1,'a':3,'g':1,'r':1,'m':1}
equal → True ✓
```
**Complexity:** O(n) time. O(k) space where k = unique elements.

---

### Template 4 — Prefix Sums

```python
prefix = [0] * (len(arr) + 1)          # prefix[0]=0 is the base case

for i in range(len(arr)):
    prefix[i+1] = prefix[i] + arr[i]   # running total up to index i

# query: sum from index i to j
range_sum = prefix[j+1] - prefix[i]    # subtract what came before i
```

**Example — sum of elements from index 1 to 2:**
```
Q1: need cumulative totals → array
Q2: key = index, value = running sum
Q3: two pass — build prefix array, then answer queries

[1,2,3,4]:
prefix = [0,1,3,6,10]
sum(1,2) = prefix[3] - prefix[1] = 6-1 = 5 ✓
```
**Complexity:** O(n) build. O(1) per query. O(n) space.

---

### Template 5 — Bucket Sort (Top K Frequent)

```python
freq = {}
for n in nums:
    freq[n] = freq.get(n, 0) + 1           # build frequency map

bucket = [[] for _ in range(len(nums) + 1)]  # index = frequency
for num, count in freq.items():
    bucket[count].append(num)               # place at frequency index

result = []
for i in range(len(bucket) - 1, 0, -1):    # right to left = most frequent first
    result.extend(bucket[i])
    if len(result) >= k:
        return result[:k]
```

**Example — top 2 most frequent elements:**
```
Q1: need counts, then rank → dict + bucket
Q2: key = element, value = count
Q3: two pass — build freq map, then place into buckets

[1,1,1,2,2,3], k=2:
freq={1:3, 2:2, 3:1}
bucket[3]=[1], bucket[2]=[2], bucket[1]=[3]
scan right → result=[1,2] → len=2=k → return [1,2] ✓
```
**Complexity:** O(n) time. O(n) space.

---
---

# 2. Two Pointers

### Mental Trigger
> *"Can I eliminate candidates by maintaining two positions that move toward each other or at different speeds?"*

**Why this works:** instead of checking every pair with a nested loop O(n²), two pointers move inward and eliminate entire ranges of candidates in one pass O(n). Requires sorted input or a naturally ordered structure.

---

### Translation Table

| You hear this... | Precondition | Pointer movement | Template |
|---|---|---|---|
| "two numbers sum to target" | sorted | toward each other | 1 |
| "palindrome", "same forwards/backwards" | any | toward each other from ends | 1 |
| "maximize area", "container with water" | any | move shorter side inward | 1 |
| "in-place removal", "no extra memory" | any | same direction, read/write | 2 |
| "three numbers sum to target" | sort first | fix one, two pointers inside | 3 |

---

### Questions to Ask Before Coding

**Q1 — Is the input sorted, or can I sort it without breaking the problem?**
- Yes → two pointers inward → Template 1 or Template 3
- No, and order matters → same direction slow/fast → Template 2

**Q2 — Do the pointers move toward each other, or in the same direction?**
- Toward each other → Template 1
- Same direction (one reads, one writes) → Template 2

**Q3 — What is the move condition?**
- This fills your `if/elif/else` inside the loop
- Sum too small → move left up
- Sum too big → move right down
- Element invalid → skip read, don't advance write

---

### Template 1 — Two Pointers Inward

```python
left, right = 0, len(arr) - 1

while left < right:
    # compute something using arr[left] and arr[right]

    if <found condition>:
        return ...
    elif <need larger value>:
        left += 1              # move left up to increase
    else:
        right -= 1             # move right down to decrease
```

**Example — which two indices sum to target=9?**
```
Q1: sorted input → valid
Q2: toward each other — comparing from both ends
Q3: sum < target → left up | sum > target → right down

[1,3,6,8,11], target=9:
left=1, right=11 → sum=12 > 9 → right--
left=1, right=8  → sum=9 == 9 → return [0,3] ✓
```
**Complexity:** O(n) time. O(1) space.

---

### Template 2 — Slow/Fast (In-place Write)

```python
write = 0                          # write pointer — next valid position

for read in range(len(arr)):
    if <element should be kept>:   # read pointer scans everything
        arr[write] = arr[read]
        write += 1                 # only advance write when keeping

return write                       # new length
```

**Example — remove all 2s in-place:**
```
Q1: order matters, no sort → same direction
Q2: same direction — read scans, write places
Q3: arr[read] != val → keep it

[1,2,2,3], val=2:
read=0(1): keep → arr[0]=1, write=1
read=1(2): skip
read=2(2): skip
read=3(3): keep → arr[1]=3, write=2
result: [1,3,...], length=2 ✓
```
**Complexity:** O(n) time. O(1) space.

---

### Template 3 — Three Sum

```python
arr.sort()                     # sort first — enables two pointers
result = []

for i in range(len(arr) - 2):
    # skip duplicate values for the fixed element
    is_duplicate = i > 0 and arr[i] == arr[i - 1]
    if is_duplicate:
        continue

    left = i + 1               # left pointer starts just after fixed element
    right = len(arr) - 1       # right pointer starts at end

    while left < right:
        total = arr[i] + arr[left] + arr[right]

        if total == 0:
            result.append([arr[i], arr[left], arr[right]])

            # skip duplicates on both sides before moving inward
            while left < right and arr[left] == arr[left + 1]:
                left += 1
            while left < right and arr[right] == arr[right - 1]:
                right -= 1

            left += 1
            right -= 1

        elif total < 0:
            left += 1          # sum too small — increase left
        else:
            right -= 1         # sum too big — decrease right

return result
```

**Example — find all triplets summing to zero:**
```
Q1: sort first → valid
Q2: fix one element, two pointers for the rest
Q3: total < 0 → left up | total > 0 → right down

[-1,0,1,2,-1,-4] → sorted: [-4,-1,-1,0,1,2]
fix -4: left=-1, right=2 → sum=-3 < 0 → left++
fix -1: left=-1, right=2 → sum=0 → found [-1,-1,2]
        left=0,  right=1 → sum=0 → found [-1,0,1]
fix  0: left=1,  right=2 → sum=3 > 0 → right--
done → [[-1,-1,2],[-1,0,1]] ✓
```
**Complexity:** O(n²) time. O(1) space excluding output.

---
---

# 3. Sliding Window

### Mental Trigger
> *"Am I looking for a subarray or substring satisfying a condition — and can I avoid recomputing from scratch when the window shifts?"*

**Why this works:** instead of recomputing the entire window each step, you adjust — add one element on the right, remove one on the left. O(n) instead of O(n²).

---

### Translation Table

| You hear this... | Variant | State | Template |
|---|---|---|---|
| "k consecutive elements", "fixed size" | Fixed | variable | 1 |
| "longest subarray/substring with condition" | Variable | set or dict | 2 |
| "shortest subarray containing all of X" | Variable | dict counter | 2 |
| "no repeating elements" | Variable | set | 2 |
| "anagram in string", "permutation in string" | Fixed | dict | 1 |

---

### Questions to Ask Before Coding

**Q1 — Fixed or variable window size?**
- Size given (e.g. k=3) → Template 1
- Find min/max length satisfying a condition → Template 2

**Q2 — What makes the window invalid?**
- This is your `while` shrink condition in Template 2
- "no duplicates" → `while arr[right] in state`
- "sum exceeds limit" → `while current_sum > limit`

**Q3 — What state do I track inside the window?**
- Running total → plain variable
- Membership → `set`
- Frequency → `dict`

---

### Template 1 — Fixed Window

```python
state = ...                            # Q3: initialise window state for first k elements
result = state

for i in range(k, len(arr)):
    # add incoming right element: arr[i]
    # remove outgoing left element: arr[i - k]
    # update result

return result
```

**Example — max sum of k=3 consecutive elements:**
```
Q1: k=3 given → fixed → Template 1
Q2: no invalidity — just slide
Q3: tracking a sum → plain variable

[2,1,5,1,3,2]:
[2,1,5]=8 → [1,5,1]=7 → [5,1,3]=9 → [1,3,2]=6
max=9 ✓
```
```python
window_sum = sum(arr[:k])
result = window_sum
for i in range(k, len(arr)):
    window_sum += arr[i]
    window_sum -= arr[i - k]
    result = max(result, window_sum)
return result
```
**Complexity:** O(n) time. O(1) space.

---

### Template 2 — Variable Window

```python
left = 0
state = ...                            # Q3: set, dict, or variable

for right in range(len(arr)):
    # expand: add arr[right] to state

    while <window is invalid>:         # Q2: your shrink condition
        # shrink: remove arr[left] from state
        left += 1

    # update result using (right - left + 1)

return result
```

**Example — longest substring with no repeating characters:**
```
Q1: find longest → variable → Template 2
Q2: invalid when duplicate → while s[right] in seen
Q3: membership → set

"abcac":
a → {a}, len=1
b → {a,b}, len=2
c → {a,b,c}, len=3
a → duplicate, shrink → {b,c,a}, len=3
c → duplicate, shrink → {a,c}, len=2
max=3 ✓
```
```python
left = 0
seen = set()
result = 0
for right in range(len(s)):
    while s[right] in seen:
        seen.remove(s[left])
        left += 1
    seen.add(s[right])
    result = max(result, right - left + 1)
return result
```
**Complexity:** O(n) time. O(k) space where k = window size.

---
---

# 4. Stack

### Mental Trigger
> *"Do I need to remember previous elements in order — and might I need to undo or revisit the most recent one?"*

**Why this works:** a stack gives O(1) access to the most recently seen element. Last in, first out — perfect for matching, nesting, or undoing in reverse order.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "matching brackets", "valid nesting" | Stack push/pop | 1 |
| "undo", "go back", "reverse order" | Stack | 1 |
| "next greater element", "daily temperatures" | Monotonic stack | 2 |
| "evaluate expression", "calculator" | Expression stack | 3 |
| "nested structure", "recursive-looking" | Stack | 1 |

---

### Questions to Ask Before Coding

**Q1 — Am I matching/validating structure, finding next greater/smaller, or evaluating an expression?**
- Matching or validating → Template 1
- Next greater/smaller element → Template 2
- Evaluate expression with operators → Template 3

**Q2 — What do I push onto the stack?**
- The element itself (for matching) → Template 1
- The index (for position-based problems) → Template 2
- Numbers and operators separately → Template 3

**Q3 — When do I pop?**
- On a closing match → Template 1
- When current element breaks the monotonic order → Template 2
- When you see an operator or closing parenthesis → Template 3

---

### Template 1 — Matching / Validation

```python
stack = []

for element in sequence:
    if <element is an opener>:
        stack.append(element)       # push — remember it for later
    elif <element is a closer>:
        if not stack or <top doesn't match>:
            return False            # nothing to match — invalid
        stack.pop()                 # matched — discard both

return len(stack) == 0             # valid only if nothing left unmatched
```

**Example — are all brackets matched in `"({[]})"` ?**
```python
stack = []
matching = {')': '(', '}': '{', ']': '['}

for char in s:
    if char in '({[':
        stack.append(char)
    elif char in ')}]':
        if not stack or stack[-1] != matching[char]:
            return False
        stack.pop()

return len(stack) == 0
```
```
Q1: matching → Template 1
Q2: push the opening bracket itself
Q3: pop when closing bracket matches top

( → push → stack=[(]
{ → push → stack=[(,{]
[ → push → stack=[(,{,[]
] → matches [ → pop → stack=[(,{]
} → matches { → pop → stack=[(]
) → matches ( → pop → stack=[]
empty → True ✓
```
**Complexity:** O(n) time. O(n) space.

---

### Template 2 — Monotonic Stack

```python
stack = []                          # stores indices of unresolved elements
result = [<default>] * len(arr)     # fill with default answer (e.g. -1 or 0)

for i in range(len(arr)):
    while stack and <current element breaks the order>:
        idx = stack.pop()
        result[idx] = <answer for idx using current element>
    stack.append(i)                 # push index — resolve it later

return result
```

**Example — next greater element for `[2,1,3,4]`:**
```python
stack = []
result = [-1] * len(arr)

for i in range(len(arr)):
    while stack and arr[i] > arr[stack[-1]]:
        idx = stack.pop()
        result[idx] = arr[i]
    stack.append(i)

return result
```
```
Q1: next element breaking order → Template 2
Q2: push index — need position to update result
Q3: pop when current > top of stack

i=0(2): stack=[0]
i=1(1): 1<2 → push → stack=[0,1]
i=2(3): 3>1 → result[1]=3, pop. 3>2 → result[0]=3, pop. push → stack=[2]
i=3(4): 4>3 → result[2]=4, pop. push → stack=[3]
result=[3,3,4,-1] ✓
```
**Complexity:** O(n) time. O(n) space.

---

### Template 3 — Expression Evaluation (any variant)

```python
stack = []

for token in sequence:

    # DECISION 1: operand — push it, use it when triggered
    if <token is a value>:
        stack.append(<parsed token>)

    # DECISION 2: operator OR closing delimiter — pop, compute, push result
    # closing ')' is a trigger to compute, same as an operator
    elif <token is an operator or closing delimiter ')'>:
        b = stack.pop()                      # most recent operand
        a = stack.pop()                      # operand before that
        stack.append(<compute(a, b, token)>) # push result — stack shrinks by 1

    # DECISION 3 (optional): opening delimiter — save current state
    # only needed when expression has scopes e.g. '('
    elif <token is '('>:
        stack.append(<current state>)        # save context, reset for inner scope

return stack[0]                              # final result sits alone on stack
```

**Example — Evaluate Reverse Polish Notation `["2","1","+","3","*"]`:**
```
Q1: evaluate expression → Template 3
Decision 1: number → push
Decision 2: operator → pop two, compute, push result
Decision 3: not needed — no delimiters in RPN

token="2" → Decision 1 → stack=[2]
token="1" → Decision 1 → stack=[2,1]
token="+" → Decision 2 → pop 1, pop 2 → 2+1=3 → stack=[3]
token="3" → Decision 1 → stack=[3,3]
token="*" → Decision 2 → pop 3, pop 3 → 3*3=9 → stack=[9]
return stack[0] = 9 ✓
```

**Example — Basic Calculator `"(1+(4+5)-3)"`:**
```python
stack = []
result = 0
sign = 1        # +1 or -1
num = 0

for char in s:
    if char.isdigit():
        num = num * 10 + int(char)      # Decision 1: build multi-digit number

    elif char in '+-':
        result += sign * num            # Decision 2: apply completed number
        num = 0
        sign = 1 if char == '+' else -1 # update sign for next number

    elif char == '(':
        stack.append(result)            # Decision 3: save result so far
        stack.append(sign)              # Decision 3: save sign before '('
        result = 0                      # reset for inner expression
        sign = 1

    elif char == ')':
        result += sign * num            # Decision 2: apply last number in brackets
        num = 0
        result *= stack.pop()           # restore sign before '('
        result += stack.pop()           # restore result before '('

result += sign * num                    # apply final number
return result
```
```
trace "(1+(4+5)-3)":
( → save result=0, sign=1 → stack=[0,1]
1 → num=1
+ → result=1, sign=+1
( → save result=1, sign=1 → stack=[0,1,1,1]
4 → num=4
+ → result=4, sign=+1
5 → num=5
) → result=4+5=9, *pop(1)=9, +pop(1)=10 → result=10
- → result=10, sign=-1
3 → num=3
) → result=10-3=7, *pop(1)=7, +pop(0)=7
return 7 ✓
```

**How other variants map:**
```
Basic Calculator — has parentheses:
  "(" → Decision 3 (save state)
  ")" → Decision 2 (restore state + apply)
  numbers/operators → Decision 1 and 2

Decode String e.g. "3[ab]":
  digit → Decision 1 (push count)
  "[" → Decision 3 (save current string)
  "]" → Decision 2 (pop and repeat)
```
**Complexity:** O(n) time. O(n) space.

---
---

# 5. Binary Search

### Mental Trigger
> *"Is the input sorted — or can I define a condition where everything to the left fails and everything to the right passes?"*

**Why this works:** each step eliminates half the remaining candidates. O(log n) instead of O(n).

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "sorted array", "find target" | Classic binary search | 1 |
| "first/last occurrence" | Keep searching after match | 1 variant |
| "minimum/maximum that satisfies condition" | Binary search on answer | 2 |
| "find broken version", "first bad commit" | Binary search on condition | 2 |
| "rotated sorted array" | Modified binary search | 1 variant |

---

### Questions to Ask Before Coding

**Q1 — Am I searching for an exact value, or a boundary?**
- Exact value → stop at first match → Template 1
- First/last occurrence → record match, keep searching → Template 1 variant
- Minimum/maximum that satisfies a condition → Template 2

**Q2 — Is the search space the input array, or the answer itself?**
- Input array → Template 1
- Answer is a number in a range → Template 2

**Q3 — What are my left and right bounds?**
- Searching array → `left=0, right=len-1`
- Searching answer space → `left=min_possible, right=max_possible`

---

### Template 1 — Classic Binary Search

```python
left, right = 0, len(arr) - 1

while left <= right:
    mid = (left + right) // 2

    if arr[mid] == target:
        return mid              # found — stop here
    elif arr[mid] < target:
        left = mid + 1          # target must be in right half
    else:
        right = mid - 1         # target must be in left half

return -1                       # not found
```

**Example — find target=7 in `[1,3,5,7,9]`:**
```python
arr = [1,3,5,7,9], target = 7
left=0, right=4

step 1: mid=2, arr[2]=5 → 5<7 → left=3
step 2: mid=3, arr[3]=7 → 7==7 → return 3 ✓
```
**Complexity:** O(log n) time. O(1) space.

---

### Template 1 variant — First/Last Occurrence

Only change: don't return on match. Record it and keep searching.

```python
left, right = 0, len(arr) - 1
result = -1

while left <= right:
    mid = (left + right) // 2

    if arr[mid] == target:
        result = mid            # record match — don't stop
        right = mid - 1         # go left for first occurrence
        # OR: left = mid + 1    # go right for last occurrence
    elif arr[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return result
```

**Example — first occurrence of 2 in `[1,2,2,2,3]`:**
```python
arr = [1,2,2,2,3], target = 2
left=0, right=4

step 1: mid=2, arr[2]=2 → match → result=2, search left → right=1
step 2: mid=0, arr[0]=1 → 1<2 → left=1
step 3: mid=1, arr[1]=2 → match → result=1, search left → right=0
step 4: left>right → stop
return 1 ✓
```
**Complexity:** O(log n) time. O(1) space.

---

### Template 2 — Binary Search on Answer

**When to use:** you're not searching an array. You're searching for the optimal value in a range of possible answers. You have a condition function that tells you if a given answer works.

```python
left, right = min_possible, max_possible

while left < right:
    mid = (left + right) // 2

    if condition(mid):          # mid works — try smaller (finding minimum)
        right = mid
    else:
        left = mid + 1          # mid doesn't work — go bigger

return left
```

**Example — Koko Eating Bananas:**

*Koko has piles of bananas. She has h hours to eat all bananas. Each hour she picks one pile and eats at most k bananas. Find the minimum k that lets her finish in time.*

```python
from math import ceil
piles = [3, 6, 7, 11], h = 8

# searching for k — the eating speed
# min possible = 1, max possible = max(piles) = 11
left, right = 1, 11

def condition(k):
    hours = sum(ceil(pile / k) for pile in piles)
    return hours <= h

step 1: mid=6  → hours=1+1+2+2=6 ≤ 8 → works → right=6
step 2: mid=3  → hours=1+2+3+4=10 > 8 → fails → left=4
step 3: mid=5  → hours=1+2+2+3=8 ≤ 8 → works → right=5
step 4: mid=4  → hours=1+2+2+3=8 ≤ 8 → works → right=4
step 5: left==right==4 → return 4 ✓
```

**The key insight:** you're not searching the array. You're asking *"does this value work?"* and binary searching the range of possible answers until you find the minimum that works.

**Complexity:** O(n log m) time where n=array size, m=answer range. O(1) space.

---
---

# 6. Linked List

### Mental Trigger
> *"Is the data sequential where each item only knows what comes next — and do I need to manipulate connections rather than values?"*

**Why this works:** linked lists have no indices. You can't jump to position 5. Every problem reduces to: which pointers do I move, and in what order?

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "reverse a sublist between positions" | Pointer reversal | 1 |
| "detect loop or cycle / find cycle entry" | Fast/slow pointers | 2 |
| "find middle without knowing length" | Fast/slow pointers | 2 |
| "Kth from end without full scan" | Two pointers gap k apart | 3 |
| "O(1) get and put with recency eviction" | Hash map + doubly linked list | 4 |

---

### Questions to Ask Before Coding

**Q1 — Am I reversing or rewiring a sublist?**
- Yes → need a `pre` anchor + pull-to-front loop → Template 1
- No → continue to Q2

**Q2 — Do I need two pointers at different speeds, or a fixed gap?**
- Different speeds → Template 2 (middle, cycle detection)
- Fixed gap k → Template 3 (kth from end)

**Q3 — Do I need O(1) lookup AND O(1) eviction by recency?**
- Yes → Template 4 (LRU Cache)

---

### Template 1 — Reverse Linked List II (LC 92)

```
Problem: reverse only the nodes between positions left and right.
Input:  1→2→3→4→5, left=2, right=4
Output: 1→4→3→2→5
```

```python
dummy = ListNode(0)
dummy.next = head
pre = dummy

for _ in range(left - 1):
    pre = pre.next              # pre stops at node just before position left

curr = pre.next                 # curr starts at the 'left' node

for _ in range(right - left):
    next_node = curr.next
    curr.next = next_node.next          # detach next_node from the chain
    next_node.next = pre.next           # next_node points to current front of reversed section
    pre.next = next_node                # pre now points to next_node (new front)
```

**Trace — Input: `1→2→3→4→5`, left=2, right=4:**
```
After walk: pre=1, curr=2
[dummy]→1→2→3→4→5
              ↑pre.next (curr)

Iteration 1: next_node=3
  curr(2).next = 4          → 1→2→4→5  (2 skips 3)
  next_node(3).next = 2     → 3→2→4→5  (3 points to old front)
  pre(1).next = 3           → 1→3→2→4→5

Iteration 2: next_node=4
  curr(2).next = 5          → 3→2→5    (2 skips 4)
  next_node(4).next = 3     → 4→3→2→5  (4 points to old front)
  pre(1).next = 4           → 1→4→3→2→5 ✓
```

**Complexity:** O(n) time. O(1) space.

---

### Template 2 — Fast/Slow Pointers

#### Part A — Find Middle

```python
slow, fast = head, head

while fast and fast.next:
    slow = slow.next                # 1 step
    fast = fast.next.next           # 2 steps

return slow                         # slow lands at middle
```

**Example — middle of `1→2→3→4→5`:**
```
STEP 1: slow=2, fast=3
STEP 2: slow=3, fast=5
STEP 3: fast.next=None → stop
return slow=3 ✓
```

---

#### Part B — Cycle Detection: Entry Point (LC 142)

```
Problem: find the node where the cycle begins. Return None if no cycle.
```

```python
# Phase 1 — detect if cycle exists; find meeting point inside cycle
slow, fast = head, head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        break
else:
    return None                     # loop finished without meeting → no cycle

# Phase 2 — find cycle entry
# Math guarantees: distance from head to entry == distance from meeting point to entry
slow = head
while slow != fast:
    slow = slow.next
    fast = fast.next

return slow                         # both pointers meet at cycle entry
```

**Why Phase 2 works:**
- Let `F` = steps from head to cycle entry, `C` = cycle length.
- When they meet inside the cycle, slow has taken `F + a` steps, fast has taken `F + a + nC` steps (lapped slow `n` times).
- Since fast moves 2× speed: `2(F + a) = F + a + nC` → `F = nC - a`.
- Reset slow to head. Both advance 1 step at a time. After `F` steps, slow is at entry. Fast started `a` steps into the cycle and has moved `F = nC - a` more steps → `a + nC - a = nC` steps into cycle → also at entry.

**Complexity:** O(n) time. O(1) space.

---

### Template 3 — Kth From End (Fixed Gap)

```python
left, right = head, head

for _ in range(k):
    right = right.next              # create gap of k

while right:
    left = left.next
    right = right.next

return left                         # left is kth from end
```

**Example — 2nd from end of `1→2→3→4→5`:**
```
After gap: left=1, right=3
STEP 1: left=2, right=4
STEP 2: left=3, right=5
STEP 3: right=None → stop
return left=4 ✓
```
**Complexity:** O(n) time. O(1) space.

---

### Template 4 — LRU Cache (LC 146)

```
Problem: design a cache with get(key) and put(key, value).
Both must run in O(1). When capacity is exceeded, evict the least recently used item.
```

#### Design

The problem needs two operations, both in O(1):
1. Look up a value by key instantly
2. Track which item was used least recently, and evict it instantly

- **Step 1 — O(1) lookup:** use a hash map `{key → node}`.
- **Step 2 — O(1) recency tracking:** doubly linked list — remove a node and reinsert at tail in constant time.
- **Step 3 — O(1) eviction:** least recently used node is always at left end. Evicting = removing first node.

```
[head] ↔ [LRU] ↔ ... ↔ [MRU] ↔ [tail]
           ↑ evict               ↑ insert/promote
```

`head` and `tail` are empty placeholder nodes that never get evicted. Without them, removing the first or last real node leaves one side with no neighbor — you'd need `if` checks every time. With them, every real node always sits between two neighbors. Remove and insert are the same two lines every time, no special cases.

#### Code

```python
class Node:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache = {}             # key → Node, for O(1) lookup

        # Empty placeholder nodes at both ends.
        # Real nodes always live between them.
        self.head = Node(0, 0)      # left boundary — LRU side
        self.tail = Node(0, 0)      # right boundary — MRU side
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        # Cut this node out by connecting its neighbors directly to each other
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_at_tail(self, node):
        # Place node just before tail — rightmost real position = most recently used
        prev = self.tail.prev       # whoever was MRU before
        prev.next = node            # old MRU now points to this node
        node.prev = prev
        node.next = self.tail
        self.tail.prev = node       # tail now points back to this node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        # Move to tail — accessing a node makes it the most recently used
        self._remove(node)
        self._insert_at_tail(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            # Key exists — remove old node before replacing it
            self._remove(self.cache[key])

        node = Node(key, value)
        self.cache[key] = node
        self._insert_at_tail(node)  # new node = most recently used

        if len(self.cache) > self.cap:
            # Evict LRU — it's always the first real node after head
            lru = self.head.next
            self._remove(lru)
            del self.cache[lru.key] # remove from map or lookup will still find it
```

#### Trace (capacity = 2)

```
put(1, 10)
Action: Insert Node(1,10). List is empty, so it goes between placeholders.
[head] ↔ [1|10] ↔ [tail]

put(2, 20)
Action: Insert Node(2,20) at tail (most recently used side).
[head] ↔ [1|10] ↔ [2|20] ↔ [tail]
          ↑ LRU              ↑ MRU

get(1)
Action: key=1 exists. Accessing it makes it MRU. Remove from middle, reinsert at tail.
[head] ↔ [2|20] ↔ [1|10] ↔ [tail]
          ↑ LRU              ↑ MRU
return 10

put(3, 30)
Action: Insert Node(3,30). Cache size = 3 > cap=2. Must evict.
LRU = head.next = Node(2,20). Remove it.
[head] ↔ [1|10] ↔ [3|30] ↔ [tail]
          ↑ LRU              ↑ MRU

get(2)
Action: key=2 was evicted. Not in cache.
return -1 ✓
```

**Complexity:** O(1) get and put. O(n) space.

---
---

# 7. Trees

### Mental Trigger
> *"Is the data hierarchical with branching parent-child relationships — and does the same logic apply at every level?"*

**Why this works:** every node is the root of its own subtree. Solve for one node assuming children are already solved — recursion handles the rest.

---

### Translation Table

| You hear this... | Template |
|---|---|
| "height", "depth", "diameter", "sum all nodes", "validate symmetry" | T1 — DFS bottom-up |
| "level by level", "layer", "same depth", "fewest steps in tree" | T2 — BFS |
| "root-to-leaf path", "path sum", "all paths" | T3 — DFS path tracking |
| "BST", "sorted tree", "LCA", "insert/search in sorted tree" | T4 — BST |

---

### Questions to Ask Before Coding

**Q1 — Do I need values from children to answer at the parent?**
- Yes → DFS bottom-up → T1
- No → continue

**Q2 — Does level (depth) matter?**
- Yes → BFS → T2

**Q3 — Do I need to track the actual path from root to leaf?**
- Yes → DFS with path → T3

**Q4 — Is the tree sorted (BST) or is the problem about ordered structure?**
- Yes → BST pattern → T4

---

### Recursion Mental Model

```
Q1. Base case:   What do I return when node is None?
Q2. Recurse:     left = dfs(node.left) | right = dfs(node.right)
Q3. Return:      What do I send up to my parent?
```

```python
def dfs(node):
    if not node: return ...           # Q1: neutral base value

    left  = dfs(node.left)            # Q2: always — trust the result
    right = dfs(node.right)

    return ...                        # Q3: what does my parent need?
```

---

**Example — Lowest Common Ancestor (LC 236)**

```
Find the deepest node that has both p and q somewhere in its subtree.

        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

p=5, q=4 → answer: 5
p=5, q=1 → answer: 3
```

```python
def lowestCommonAncestor(self, root, p, q):
    def dfs(node):
        if not node: return None          # Q1: None = "found nothing here"

        left  = dfs(node.left)            # Q2
        right = dfs(node.right)           # Q2

        if node == p or node == q:        # Q3a: I am a target — report myself up
            return node
        if left and right:                # Q3b: both sides found something — I am LCA
            return node
        return left or right              # Q3c: bubble up whichever side found something

    return dfs(root)
```

**Trace — p=5, q=4:**

```
dfs(3)
  dfs(3).left = dfs(5)
    dfs(5).left = dfs(6)
      left=None, right=None. Not a target. → return None

    dfs(5).right = dfs(2)
      dfs(2).left = dfs(7)
        left=None, right=None. Not a target. → return None

      dfs(2).right = dfs(4)
        left=None, right=None. node==q. → return node(4)

      left=None, right=node(4). Not a target. One side found. → return node(4)

    left=None, right=node(4). node==p. → return node(5)

  dfs(3).right = dfs(1)
    dfs(1).left = dfs(0)
      left=None, right=None. Not a target. → return None

    dfs(1).right = dfs(8)
      left=None, right=None. Not a target. → return None

    left=None, right=None. Not a target. → return None

  left=node(5), right=None. One side found. → return node(5) ✓
```

The LCA is never found on the way down. It emerges on the way back up.

---

### DFS vs BFS

| | DFS | BFS |
|---|---|---|
| Structure | Recursion (call stack) | Queue |
| Direction | Dives deep first | Widens level by level |
| Space | O(h) — tree height | O(w) — widest level |
| Use when | Answer lives deep in a subtree | Need closest/shortest/first at any level |
| Example problems | Max depth, diameter, path sum, validate BST | Level order, min depth, right side view |

**Trigger words:**
- DFS → "height", "depth", "path", "subtree", "diameter", "validate"
- BFS → "level by level", "layer", "minimum depth", "nearest", "closest"

**Rule of thumb:** answer could be anywhere deep → DFS. Need the closest/shortest/first at any level → BFS.

---

### Traversals

```
        1
       / \
      2   3
     / \   \
    4   5   6
   /       /
  7       8
```

---

**Preorder — node first, then children (top-down)**

Visit the node before going deeper. You see parents before their children.

```python
def preorder(node):
    if not node: return

    result.append(node.val)   # visit node FIRST
    preorder(node.left)       # then left subtree
    preorder(node.right)      # then right subtree
```

```
Output: 1 → 2 → 4 → 7 → 5 → 3 → 6 → 8
```

---

**Inorder — left first, then node, then right**

Dive all the way left before visiting anything. For BSTs this gives sorted order.

```python
def inorder(node):
    if not node: return

    inorder(node.left)        # left subtree first
    result.append(node.val)   # visit node IN BETWEEN
    inorder(node.right)       # then right subtree
```

```
Output: 7 → 4 → 2 → 5 → 1 → 3 → 8 → 6
```

---

**Postorder — children first, then node (bottom-up)**

Fully resolve both subtrees before visiting the node. You see children before parents.

```python
def postorder(node):
    if not node: return

    postorder(node.left)      # left subtree first
    postorder(node.right)     # right subtree second
    result.append(node.val)   # visit node LAST
```

```
Output: 7 → 4 → 5 → 2 → 8 → 6 → 3 → 1
```

---

### T1 — DFS Recursive (Bottom-Up)

**Why T1 exists:** Many tree problems (height, sum, balance, diameter) need the answer from both subtrees before a node can answer. DFS dives to the bottom first, then answers flow back up.

**Template**

```python
def dfs(node):
    # Q1: Base case — what do I return when there's no node?
    #     A neutral value that won't affect the parent's calculation
    if not node:
        return 0           # 0 for height/sum, True for validation, None for search

    # Q2: Recurse — let both subtrees solve themselves first
    left  = dfs(node.left)
    right = dfs(node.right)

    # Q3: Return — combine what the children gave you, send it up
    return 1 + max(left, right)   # height: this node adds 1 to the taller side
                                   # sum:    node.val + left + right
                                   # valid:  left and right and <condition>

dfs(root)
```

**Why the code looks like this**

- **Recurse before returning:** A node can't answer until its children have answered.
- **Neutral base value:** An empty subtree must not change the parent's answer (height 0, sum 0).
- **Return, don't store:** Each node hands its answer to its parent. The parent never looks deeper.

**Example — height of tree**

```
Tree:
        1
       / \
      2   3
     / \   \
    4   5   6
   /
  7

Trace:
dfs(1) → dfs(2) → dfs(4) → dfs(7)    # dive to the deepest node first

dfs(7) → left=0, right=0 → return 1
dfs(4) → left=1, right=0 → return 2
dfs(5) → left=0, right=0 → return 1
dfs(2) → left=2, right=1 → return 3
dfs(6) → left=0, right=0 → return 1
dfs(3) → left=0, right=1 → return 2
dfs(1) → left=3, right=2 → return 4  ✓
```

**Complexity:** O(n) time. O(h) space (call stack depth = tree height).

---

### T2 — BFS (Level by Level)

**Why T2 exists:** Some problems care about levels, not depth in a subtree: "level by level", "closest", "shortest", "first at any level".

**The idea**

- **Why a queue?** A queue is FIFO (first in, first out). Nodes are processed in the order they were discovered, so a whole level is finished before the next one starts. A stack (LIFO) would dive deep instead.
- **Algorithm in short:**
  1. Put root in the queue
  2. Snapshot the queue size — that is exactly one level
  3. Process that many nodes, adding their children to the queue
  4. Children form the next level. Repeat

**Template**

```python
from collections import deque

def bfs(root):
    if not root:
        return []

    queue = deque([root])           # seed with root — this is level 0
    result = []

    while queue:                    # one loop = one full level
        level_size = len(queue)     # snapshot: exactly this many nodes are in the current level
                                    # (children are not added yet, so the count is clean)
        level = []

        for _ in range(level_size): # process this level only — no more, no less
            node = queue.popleft()  # FIFO: oldest discovery goes first
            level.append(node.val)

            if node.left:  queue.append(node.left)    # children wait in the queue
            if node.right: queue.append(node.right)   # for the NEXT level

        result.append(level)        # one full level collected

    return result
```

**Why the code looks like this**

- **`level_size = len(queue)` before the loop:** Children are added during the loop. Taking the size first stops the loop from running into the next level.
- **`popleft()`:** Takes the oldest node. That keeps left-to-right order within a level.
- **Children appended to the back:** They sit behind the current level, so they are only reached after it finishes.

**Example — level-order traversal**

```
Tree:
        1
       / \
      2   3
     / \   \
    4   5   6
   /       /
  7       8

Trace:
Start: queue=[1], result=[]

Level 1: level_size=1, queue=[1]
  pop 1 → add 2,3     → level=[1]
  result=[[1]]

Level 2: level_size=2, queue=[2,3]
  pop 2 → add 4,5
  pop 3 → add 6       → level=[2,3]
  result=[[1],[2,3]]

Level 3: level_size=3, queue=[4,5,6]
  pop 4 → add 7
  pop 5 → no children
  pop 6 → add 8       → level=[4,5,6]
  result=[[1],[2,3],[4,5,6]]

Level 4: level_size=2, queue=[7,8]
  pop 7, pop 8        → level=[7,8]
  result=[[1],[2,3],[4,5,6],[7,8]] ✓
```

**Complexity:** O(n) time. O(w) space (queue holds at most the widest level).

---

### T3 — DFS with Path Tracking

**Why T3 exists:** Some problems ask about the **route** from the root, not just a value. A node can't see what's above it, so we carry the route down with us.

**How is T3 different from T1?**

| | T1 | T3 |
|---|---|---|
| Direction | Bottom-up | Top-down |
| What you pass | Nothing extra | `path` and running state |
| When you act | After children return | Before recursing |
| Goal | Compute something at every node | Track the route from root to leaf |

**Template**

```python
result = []

def dfs(node, path):
    if not node:
        return                          # nothing to do past a leaf

    # Step in — act before recursing
    path.append(node.val)

    # Leaf = end of a root-to-leaf route
    if not node.left and not node.right:
        if <condition on path>:         # ← the only part that changes per problem
            result.append(path[:])      # save a snapshot of the path

    # Go deeper
    dfs(node.left,  path)
    dfs(node.right, path)

    # Step out — undo before returning to parent (backtracking)
    path.pop()

dfs(root, [])
return result
```

If the condition needs a running value (like a sum), pass it as an extra argument: `dfs(node.left, path, current_sum)`. Ints are copied on each call, so they need no undo. Only the list needs `pop()`.

**Why the code looks like this**

- **`path.append` before recursing:** The route is built on the way down, so add yourself first.
- **`path.pop()` at the end:** One list is shared by the whole tree. Remove yourself when leaving, or the next branch inherits your node.
- **`path[:]` when saving:** `path` keeps changing. A copy freezes what it looked like at this leaf.

**Example**

**Problem:** Every root-to-leaf route has a total (add up the values along it). Return the routes whose total equals 8.

Here the template is filled in for this problem. The extra argument is `current_sum` and the condition is `current_sum == target`.

```python
result = []

def dfs(node, path, current_sum):
    if not node:
        return

    path.append(node.val)
    current_sum += node.val

    if not node.left and not node.right:
        if current_sum == target:       # ← the template's <condition>
            result.append(path[:])

    dfs(node.left,  path, current_sum)
    dfs(node.right, path, current_sum)

    path.pop()

dfs(root, [], 0)
return result
```

```
Tree:
        1
       / \
      2   6
     / \   \
    4   5   3
   /
  7

target = 8

Trace:
dfs(1, [], 0)            → path=[1], sum=1
  dfs(2, [1], 1)         → path=[1,2], sum=3
    dfs(4, [1,2], 3)     → path=[1,2,4], sum=7
      dfs(7,[1,2,4],7)   → path=[1,2,4,7], sum=14 → leaf, sum≠8 → skip
      pop 7 → path=[1,2,4]
    pop 4 → path=[1,2]
    dfs(5, [1,2], 3)     → path=[1,2,5], sum=8 → leaf, sum==8 → add [1,2,5] ✓
    pop 5 → path=[1,2]
  pop 2 → path=[1]
  dfs(6, [1], 1)         → path=[1,6], sum=7
    dfs(3, [1,6], 7)     → path=[1,6,3], sum=10 → leaf, sum≠8 → skip
    pop 3 → path=[1,6]
  pop 6 → path=[1]
pop 1 → path=[]

result = [[1,2,5]] ✓
```

**Complexity:** O(n) time. O(h) space (path length = tree height at most).

---

### T4 — BST Problems

**Why T4 exists:** A BST is sorted by structure. At every node, left is smaller and right is larger. Every BST problem uses this to either **discard one side** or **read values in sorted order**. The 3 blocks below cover most FAANG BST questions.

```
Tree used in the traces:
        8
       / \
      4   12
     / \    \
    2   6    14
       /
      5
```

#### Block 1: Go one side only

```python
def dfs(node, target):
    if not node:
        return ...                          # fell off: not found / empty slot

    if <node is the answer>:
        return node

    if target < node.val:
        return dfs(node.left, target)       # smaller → can only be on the left
    return dfs(node.right, target)          # larger → can only be on the right
```

- **One recursive call, not two:** the property tells you which side to take.
- **Per problem, only the base case and the "answer" check change.**

**Unlocks:** search, insert, LCA of a BST, delete, trim, closest value, floor/ceiling.

**Trace (search 5):** `8 → left → 4 → right → 6 → left → 5 ✓` (4 nodes visited out of 7)

#### Block 2: Inorder is sorted

Inorder (left → node → right) visits values in **sorted order**.

```python
def inorder(root):
    stack, node = [], root
    while stack or node:
        while node:                 # go as far left as possible
            stack.append(node)
            node = node.left
        node = stack.pop()          # smallest unvisited node
        visit(node)                 # ← the problem's work goes here
        node = node.right           # then its right subtree
```

- **Why iterative:** you can stop after any `visit` and resume later. That is exactly BST Iterator, and it also makes "stop at the kth" cheap.
- **Recursive version:** `inorder(left) → visit(node) → inorder(right)`.

**Unlocks:** kth smallest, validate (must be strictly increasing), successor/predecessor, min difference, recover BST (find where the order breaks), two-sum in a BST, BST iterator.

**Trace:** `2 → 4 → 5 → 6 → 8 → 12 → 14` ✓ sorted

#### Block 3: Pass a range down

**Why not compare with the parent only?** The rule applies to the whole subtree. A node can look fine against its parent and still break an ancestor's rule.

```python
def dfs(node, low, high):                       # every node here must stay inside (low, high)
    if not node:
        return True
    if not (low < node.val < high):
        return False
    return (dfs(node.left,  low, node.val) and  # going left: upper bound becomes the parent
            dfs(node.right, node.val, high))    # going right: lower bound becomes the parent

dfs(root, float('-inf'), float('inf'))
```

**Unlocks:** validate BST, construct BST from preorder (take the next value only if it fits the range), serialize/deserialize BST (the range tells you where each node belongs, so no null markers).

**Trace (invalid tree: 5 → left 3 → right child 6):**
```
dfs(5, -inf, inf) ✓ → dfs(3, -inf, 5) ✓ → dfs(6, 3, 5): 6 < 5? No → False ✗
```
6 passes the parent check (6 > 3) but breaks the ancestor's rule (must be < 5).

**Complexity:** Block 1: O(h). Blocks 2, 3: O(n). Balanced tree → h = log n. Skewed tree → h = n.

---
---

# 8. Heap / Priority Queue

### Mental Trigger
> *"Do I need to repeatedly find the smallest or largest element from a changing collection?"*

**Why this works:** a heap gives O(1) access to min or max, and O(log n) insert. Better than sorting when you only need the top K or a running extreme.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "top K", "K largest", "K most frequent" | Min heap size K | 1 |
| "K smallest" | Max heap size K | 1 variant |
| "running median" | Two heaps (max + min) | 2 |
| "merge K sorted lists" | Min heap with K elements | 3 |
| "next task by priority" | Min/max heap | 1 |

---

### Questions to Ask Before Coding

**Q1 — Do I need top K largest, or top K smallest?**
- Top K largest → min heap of size K (eject smallest) → Template 1
- Top K smallest → max heap of size K (eject largest, negate values) → Template 1 variant

**Q2 — Is the collection static (given upfront) or dynamic (streaming)?**
- Static → heapify once, pop K times
- Dynamic/streaming → maintain heap as elements arrive → Template 1

**Q3 — Do I need a running median, or merge multiple sorted sources?**
- Running median → two heaps → Template 2
- Merge K sorted lists → min heap with one element per list → Template 3

---

### Template 1 — Top K Largest (Min Heap)

```python
import heapq

heap = []
for num in nums:
    heapq.heappush(heap, num)       # push element
    if len(heap) > k:
        heapq.heappop(heap)         # eject smallest — keep only top K

return heap                         # K largest elements remain
```

**Example — top 3 largest from `[3,1,4,1,5,9,2,6]`:**
```
Q1: top K largest → min heap → Template 1
Q2: static input

push 3→[3], push 1→[1,3], push 4→[1,3,4]
push 1→pop 1→[1,3,4], push 5→pop 1→[3,4,5]
push 9→pop 3→[4,5,9], push 2→pop 2→[4,5,9]
push 6→pop 4→[5,6,9] ✓
```
**Complexity:** O(n log k) time. O(k) space.

---

### Template 2 — Running Median (Two Heaps)

```python
import heapq

small = []      # max heap (negate) — lower half
large = []      # min heap — upper half

def add_num(num):
    heapq.heappush(small, -num)

    if small and large and (-small[0] > large[0]):
        heapq.heappush(large, -heapq.heappop(small))

    if len(small) > len(large) + 1:
        heapq.heappush(large, -heapq.heappop(small))
    if len(large) > len(small):
        heapq.heappush(small, -heapq.heappop(large))

def find_median():
    if len(small) > len(large):
        return -small[0]
    return (-small[0] + large[0]) / 2
```

**Example — median after each number arrives:**
```
Q3: running median → two heaps → Template 2
Lower half in max heap, upper half in min heap
Median is at the boundary between them
```
**Complexity:** O(log n) per insert. O(1) per median query.

---

### Template 3 — Merge K Sorted Lists

```python
import heapq

heap = []
for i, lst in enumerate(lists):
    if lst:
        heapq.heappush(heap, (lst[0], i, 0))   # (value, list_idx, elem_idx)

result = []
while heap:
    val, i, j = heapq.heappop(heap)            # global minimum
    result.append(val)
    if j + 1 < len(lists[i]):
        heapq.heappush(heap, (lists[i][j+1], i, j+1))

return result
```

**Example — merge `[[1,4,7],[2,5,8],[3,6,9]]`:**
```
Q3: merge K sorted → Template 3

initial heap=[(1,0,0),(2,1,0),(3,2,0)]
pop 1 → result=[1], push (4,0,1)
pop 2 → result=[1,2], push (5,1,1)
pop 3 → result=[1,2,3], push (6,2,1)
...continues → [1,2,3,4,5,6,7,8,9] ✓
```
**Complexity:** O(n log k) time where k=number of lists.

---
---

# 9. Backtracking

### Mental Trigger
> *"Do I need to list every valid combination, arrangement or path?"*

**Idea:** try a choice, go deeper, undo it, try the next. Skip a choice the moment it can't work.

---

### The Template

```python
result = []

def backtrack(<state>, path):
    if <goal reached>:                      # Q3
        result.append(path[:])              # save a copy of the path
        return

    for choice in <choices>:                # Q1
        if <not allowed>:                   # Q4
            continue                        # skip this choice
        path.append(choice)                 # choose
        backtrack(<next state>, path)       # recurse (Q2)
        path.pop()                          # undo

backtrack(<start state>, [])
return result
```

**Why the code looks like this**
- **`for` loop:** tries each choice, one at a time.
- **choose → recurse → undo:** add a choice, go deeper, then remove it so the next choice starts clean.
- **`path[:]`:** `path` keeps changing as we go, so we store a copy of it.

---

### Filling the Loop: 3 Slots

Every backtracking loop has the same skeleton. Three slots change from problem to problem:

```python
for i in ___A___:                  # how far can I look?
    if ___B___:                    # which choices do I skip?
        continue
    path.append(nums[i])
    backtrack(___C___)             # what do I pass on?
    path.pop()
```

Fill them in with 2 steps. **Do Step 1 first. Step 2 depends on its answer.**

---

#### Step 1: Does order matter? (Is `[1,2]` different from `[2,1]`?)

| Answer | Slot A | Slot B | Slot C |
|---|---|---|---|
| **No** (subsets, combinations) | `range(start, len(nums))` | nothing yet | `i + 1` |
| **Yes** (permutations) | `range(len(nums))` | `used[i]` | no `start` |

- **No:** look only **forward**, so each group is built in one order only.
- **Yes:** look at **everything**, and use `used` so a position isn't picked twice.
- **Can a number be reused?** Pass `i` instead of `i + 1` (Combination Sum).

---

#### Step 2: Does the input have duplicate values? (like `[1,2,1]`)

- **No:** you're done, nothing to add.
- **Yes:** **sort first**, then add one more skip line to Slot B. Which line depends on your Step 1 answer:

| Step 1 answer | Add to Slot B |
|---|---|
| **Order doesn't matter** | `if i > start and nums[i] == nums[i-1]: continue` |
| **Order matters** | `if i > 0 and nums[i] == nums[i-1] and not used[i-1]: continue` |

**In plain words:**
- **Order doesn't matter:** "If I'm not the first pick in this loop and I equal the number before me, skip me."
- **Order matters:** "If I equal the number before me, and that one isn't in the path right now, skip me."

---

#### Quick map

| Problem | Order matters? | Duplicates in input? | Extra Slot B line |
|---|---|---|---|
| Subsets | No | No | none |
| Subsets II | No | Yes | `i > start` version |
| Combination Sum II | No | Yes | `i > start` version |
| Permutations | Yes | No | none (just `used[i]`) |
| Permutations II | Yes | Yes | `i > 0 … not used[i-1]` version |

---

### The 4 Questions

| # | Question | Fills |
|---|---|---|
| Q1 | What can I choose at each step? | `<choices>` |
| Q2 | What do I need to remember after choosing? | `<state>` |
| Q3 | When is the path complete? | `<goal reached>` |
| Q4 | When should I skip a choice? | `<not allowed>` |

---

### Example 1: Combination Sum

**Problem:** Pick numbers from `[2,3,6,7]`, reusing allowed, that add up to 7. Return all combinations.

| Question | Answer | Goes into the code |
|---|---|---|
| Q1: Choices | Numbers from `start` onward (order doesn't matter, reuse is fine) | `for i in range(start, len(candidates))` |
| Q2: Remember | `start` and how much is `remaining` | `backtrack(i, path, remaining - candidates[i])` |
| Q3: Goal | Nothing remaining | `remaining == 0` |
| Q4: Skip | A number bigger than what's remaining | `candidates[i] > remaining` |

```python
def combinationSum(candidates, target):
    result = []
    candidates.sort()

    def backtrack(start, path, remaining):
        if remaining == 0:                          # Q3: goal
            result.append(path[:])                  # save a copy of the path
            return

        for i in range(start, len(candidates)):     # Q1: numbers from start onward
            if candidates[i] > remaining:           # Q4: too big → stop (sorted, so later ones are bigger too)
                break
            path.append(candidates[i])              # choose
            backtrack(i, path, remaining - candidates[i])   # recurse: i (not i+1) lets us reuse a number
            path.pop()                              # undo

    backtrack(0, [], target)
    return result
```

**Complexity:** O(n^(target ÷ smallest candidate)) time, as an upper bound. O(target ÷ smallest candidate) space for the recursion depth.

---

### Example 2: Word Search

**Problem:** Is `"CAT"` in the grid, moving up/down/left/right, never reusing a cell?

```
Grid:
  C A T
  A B T
```

| Question | Answer | Goes into the code |
|---|---|---|
| Q1: Choices | The 4 neighbor cells | `for dr, dc in ((1,0), (-1,0), (0,1), (0,-1))` |
| Q2: Remember | Position `(r, c)` and how many letters matched (`k`) | `backtrack(r + dr, c + dc, k + 1)` |
| Q3: Goal | All letters matched | `k == len(word)` |
| Q4: Skip | Off the grid, wrong letter, or already used | the `if` at the top |

Two differences from the template:
- There is no `path` list. Marking cells on the grid is the path, so **choose = mark the cell** and **undo = restore it**.
- The question is yes/no ("is it there?"), so it returns `True`/`False` instead of a list. The skip check also sits at the top of the call instead of inside the loop. It's the same idea.

```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def backtrack(r, c, k):
        if k == len(word):                          # Q3: goal
            return True
        if r < 0 or r >= rows or c < 0 or c >= cols or board[r][c] != word[k]:
            return False                            # Q4: off grid, wrong letter, or used

        saved = board[r][c]
        board[r][c] = '#'                           # choose: mark the cell as used
        found = any(backtrack(r + dr, c + dc, k + 1)            # recurse
                    for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)))   # Q1: 4 neighbors
        board[r][c] = saved                         # undo: put the letter back
        return found

    for r in range(rows):
        for c in range(cols):
            if backtrack(r, c, 0):                  # try every cell as the start
                return True
    return False
```

```
Trace (neighbor order: down, up, right, left):
bt(0,0,0): 'C' = word[0] ✓ → mark
  down  (1,0)='A' = word[1] ✓ → bt(1,0,1): mark
    down  off grid → skip
    up    (0,0) is '#' → skip
    right (1,1)='B' ≠ 'T' → skip
    left  off grid → skip
    all four failed → dead end → restore 'A' at (1,0)
  up    off grid → skip
  right (0,1)='A' = word[1] ✓ → bt(0,1,1): mark
    down  (1,1)='B' ≠ 'T' → skip
    up    off grid → skip
    right (0,2)='T' = word[2] ✓ → bt(0,2,2): mark
      neighbor call has k = 3 = len(word) → True ✓
found → True passes up, restoring cells on the way

result = True ✓
```

**Complexity:** O(rows·cols·3^L) time, where L = word length (3, not 4, because you never step back to the cell you came from). O(L) space.

---

### Variation

- **Yes/no question ("does one exist?")?** Return `True` at the goal, and stop as soon as a call returns `True` (Word Search above).

---
---

# 10. Graphs

### Mental Trigger
> *"Is the data a network of connected nodes where I need to traverse connections or find paths?"*

**Why this works:** DFS explores depth first — good for connectivity and components. BFS explores level by level — good for shortest paths. Both use a visited set to avoid revisiting.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "connected components", "count islands" | DFS/BFS + visited | 1 |
| "shortest path", "fewest steps" | BFS | 2 |
| "can reach", "is path possible" | DFS/BFS | 1 or 2 |
| "dependency order", "valid ordering" | Topological sort | 3 |
| "spread", "infection", "multi-source" | Multi-source BFS | 2 variant |
| "union", "merge groups", "same component" | Union-Find | 4 |

---

### Questions to Ask Before Coding

**Q1 — Do I need shortest path, or just reachability/components?**
- Shortest path → BFS → Template 2
- Reachability or count components → DFS → Template 1

**Q2 — Is there a dependency ordering (A must come before B)?**
- Yes → Topological sort → Template 3

**Q3 — Is the graph a grid or an adjacency list?**
- Grid → DFS with bounds checking → Template 1 grid variant
- Adjacency list → standard DFS/BFS with visited set

**Q4 — Am I repeatedly merging groups and checking if two nodes are in the same group?**
- Yes → Union-Find → Template 4 (faster than repeated DFS for dynamic connectivity)

---

### Template 1 — DFS (Components / Reachability)

```python
def dfs(node, visited):
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(neighbor, visited)

visited = set()
dfs(start, visited)
```

**Grid variant — count islands:**
```python
def dfs(r, c):
    if r < 0 or r >= rows or c < 0 or c >= cols:
        return
    if grid[r][c] != '1':
        return
    grid[r][c] = '0'               # mark visited
    dfs(r+1,c); dfs(r-1,c); dfs(r,c+1); dfs(r,c-1)

count = 0
for r in range(rows):
    for c in range(cols):
        if grid[r][c] == '1':
            dfs(r, c)
            count += 1
return count
```

**Example — count islands in `[[1,1,0],[1,0,0],[0,0,1]]`:**
```
Q1: count components → DFS → Template 1
Q3: grid → use bounds checking

(0,0)=1 → DFS marks (0,0),(0,1),(1,0) → count=1
(2,2)=1 → DFS marks (2,2) → count=2 ✓
```
**Complexity:** O(V+E) time. O(V) space.

---

### Template 2 — BFS (Shortest Path)

```python
from collections import deque

queue = deque([start])
visited = {start}
distance = 0

while queue:
    for _ in range(len(queue)):     # process one full level
        node = queue.popleft()
        if node == target:
            return distance
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    distance += 1

return -1                           # not reachable
```

**Example — fewest steps from A to C:**
```
Q1: shortest path → BFS → Template 2

queue=[A], distance=0
pop A → add B,D → queue=[B,D], distance=1
pop B → add C → queue=[D,C], distance=2
pop C → target! → return 2 ✓
```
**Complexity:** O(V+E) time. O(V) space.

---

### Template 3 — Topological Sort

```python
from collections import deque

in_degree = {node: 0 for node in graph}
for node in graph:
    for neighbor in graph[node]:
        in_degree[neighbor] += 1

queue = deque([n for n in in_degree if in_degree[n] == 0])
order = []

while queue:
    node = queue.popleft()
    order.append(node)
    for neighbor in graph[node]:
        in_degree[neighbor] -= 1
        if in_degree[neighbor] == 0:
            queue.append(neighbor)

return order if len(order) == len(graph) else []  # empty = cycle
```

**Example — valid build order for A→C, B→C, C→D:**
```
Q2: dependency ordering → Template 3

in_degree: A=0, B=0, C=2, D=1
queue=[A,B]
pop A → C in_degree→1
pop B → C in_degree→0 → add C
pop C → D in_degree→0 → add D
pop D → order=[A,B,C,D] ✓
```
**Complexity:** O(V+E) time. O(V) space.

---

### Template 4 — Union-Find (Disjoint Set)

```python
parent = list(range(n))             # each node starts as its own parent
rank   = [0] * n                    # rank for union by rank optimisation

def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x]) # path compression
    return parent[x]

def union(x, y):
    px, py = find(x), find(y)
    if px == py:
        return False                 # already in same component
    if rank[px] < rank[py]:
        px, py = py, px
    parent[py] = px                  # attach smaller tree under larger
    if rank[px] == rank[py]:
        rank[px] += 1
    return True
```

**Example — count connected components among n=4 nodes, edges=[(0,1),(1,2)]:**
```
Q4: repeatedly merging groups → Union-Find → Template 4

parent=[0,1,2,3]
union(0,1) → parent=[0,0,2,3], components=3
union(1,2) → find(1)=0, parent=[0,0,0,3], components=2
Node 3 isolated → 2 components ✓
```
**Complexity:** O(α(n)) per operation where α is inverse Ackermann — effectively O(1).

---
---

# 11. Dynamic Programming

### Mental Trigger
> *"Am I solving the same subproblem multiple times — and can I build the answer from smaller answers I've already computed?"*

**Why this works:** store answers to subproblems. Never recompute. Turns exponential recursion into polynomial DP.

---

### Translation Table

| You hear this... | Sub-pattern | Template |
|---|---|---|
| "number of ways", "count paths", "climb stairs" | 1D DP | 1 |
| "max/min value", "include or exclude each item" | 0/1 Knapsack | 2 |
| "minimum coins", "items reusable" | Unbounded knapsack | 3 |
| "two sequences", "edit distance", "LCS" | 2D DP | 4 |
| "palindrome", "split string optimally" | 2D DP on string | 4 variant |
| "longest increasing subsequence", "chain of pairs" | LIS — 1D DP with inner loop | 5 |

---

### Questions to Ask Before Coding

**Q1 — What is the subproblem?**
- This defines your dp array index
- "ways to reach step i" → `dp[i]` → Template 1
- "max value with capacity w" → `dp[w]` → Template 2 or 3
- "edit distance for s1[:i] and s2[:j]" → `dp[i][j]` → Template 4
- "longest increasing subsequence ending at i" → `dp[i]` → Template 5

**Q2 — Can items/choices be reused?**
- No reuse → iterate backwards over capacity → Template 2
- Reuse allowed → iterate forwards → Template 3

**Q3 — What are the base cases?**
- Always set these first — they're the foundation everything builds on
- `dp[0] = 0` (zero amount needs zero coins)
- `dp[0][j] = j` (delete j chars to match empty string)
- `dp[i] = 1` (every element is a subsequence of length 1) → Template 5

---

### Template 1 — 1D DP (Count Ways)

```python
dp = [0] * (n + 1)
dp[0] = 1                           # Q3: base case
dp[1] = 1

for i in range(2, n + 1):
    dp[i] = dp[i-1] + dp[i-2]      # Q1: subproblem depends on previous two

return dp[n]
```

**Example — ways to climb n=5 stairs:**
```
Q1: dp[i] = ways to reach step i
Q3: dp[0]=1, dp[1]=1

dp[2]=2, dp[3]=3, dp[4]=5, dp[5]=8 ✓
```
**Complexity:** O(n) time. O(n) space (reducible to O(1)).

---

### Template 2 — 0/1 Knapsack (No reuse)

```python
dp = [0] * (capacity + 1)

for weight, value in items:
    for w in range(capacity, weight - 1, -1):   # Q2: backwards = no reuse
        dp[w] = max(dp[w], dp[w - weight] + value)

return dp[capacity]
```

**Example — max value with capacity=5, items=[(2,3),(3,4),(4,5)]:**
```
Q1: dp[w] = max value achievable with capacity w
Q2: no reuse → iterate backwards
Q3: dp[0]=0 (zero capacity, zero value)

after (2,3): dp[5]=3
after (3,4): dp[5]=max(3, dp[2]+4)=7
return 7 ✓
```
**Complexity:** O(n * capacity) time. O(capacity) space.

---

### Template 3 — Unbounded Knapsack (Reuse allowed)

```python
dp = [float('inf')] * (amount + 1)
dp[0] = 0                           # Q3: base case

for coin in coins:
    for w in range(coin, amount + 1):   # Q2: forwards = reuse allowed
        dp[w] = min(dp[w], dp[w - coin] + 1)

return dp[amount] if dp[amount] != float('inf') else -1
```

**Example — minimum coins for amount=6, coins=[1,2,5]:**
```
Q1: dp[w] = min coins to make amount w
Q2: reuse allowed → iterate forwards
Q3: dp[0]=0

after coin 1: dp=[0,1,2,3,4,5,6]
after coin 2: dp=[0,1,1,2,2,3,3]
after coin 5: dp=[0,1,1,2,2,1,2]
return dp[6]=2 ✓
```
**Complexity:** O(amount * coins) time. O(amount) space.

---

### Template 4 — 2D DP (Two Sequences)

```python
dp = [[0] * (len(s2) + 1) for _ in range(len(s1) + 1)]

# Q3: base cases — usually first row/col
for i in range(len(s1) + 1): dp[i][0] = i
for j in range(len(s2) + 1): dp[0][j] = j

for i in range(1, len(s1) + 1):
    for j in range(1, len(s2) + 1):
        if s1[i-1] == s2[j-1]:
            dp[i][j] = dp[i-1][j-1]        # characters match
        else:
            dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])

return dp[len(s1)][len(s2)]
```

**Example — edit distance between "cat" and "cut":**
```
Q1: dp[i][j] = edit distance for s1[:i] and s2[:j]
Q3: dp[i][0]=i (delete i chars), dp[0][j]=j

c vs c: match → dp[1][1]=0
a vs u: no match → 1+min(dp[0][1],dp[1][0],dp[0][0])=1
t vs t: match → dp[3][3]=1 ✓
```
**Complexity:** O(n*m) time. O(n*m) space.

---

### Template 5 — LIS (Longest Increasing Subsequence)

```python
dp = [1] * len(nums)               # Q3: every element alone = length 1

for i in range(len(nums)):
    for j in range(i):             # check all previous elements
        if nums[j] < nums[i]:      # nums[i] can extend subsequence ending at j
            dp[i] = max(dp[i], dp[j] + 1)

return max(dp)
```

**Example — LIS of `[10, 9, 2, 5, 3, 7, 101, 18]`:**
```
Q1: dp[i] = length of LIS ending at index i
Q3: dp[i]=1 (each element is its own subsequence)

dp[0]=1 (10)
dp[1]=1 (9, can't extend 10)
dp[2]=1 (2, can't extend anything)
dp[3]=2 (5 extends 2 → [2,5])
dp[4]=2 (3 extends 2 → [2,3])
dp[5]=3 (7 extends [2,5] or [2,3] → [2,5,7] or [2,3,7])
dp[6]=4 (101 extends [2,5,7] → [2,5,7,101])
dp[7]=4 (18 extends [2,5,7] → [2,5,7,18])
max(dp) = 4 ✓
```
**Complexity:** O(n²) time. O(n) space.

---
---

# 12. Greedy

### Mental Trigger
> *"Can I make the locally best choice at each step without reconsidering — and does that guarantee the global best?"*

**Why this works:** greedy skips the need to explore all possibilities. It only works when local optimal choices don't conflict with global optimality — which you should verify before committing.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "maximum meetings/events to attend" | Sort by end time, pick greedily | 1 |
| "can you reach the end" | Track max reachable index | 2 |
| "minimum platforms/resources needed" | Sort + count overlaps | 1 variant |
| "assign jobs to minimize time" | Sort by duration | 1 variant |

---

### Questions to Ask Before Coding

**Q1 — What is the locally optimal choice at each step?**
- Earliest ending interval → maximizes future options → Template 1
- Furthest reachable position → maximizes progress → Template 2

**Q2 — Does making the local optimal choice ever hurt us globally?**
- If yes → greedy won't work, need DP
- If no → greedy is valid, proceed

**Q3 — What do I sort by?**
- End time → for scheduling/interval problems → Template 1
- Start time → for resource counting problems
- Value/weight ratio → for fractional knapsack

---

### Template 1 — Interval Scheduling (Max Non-overlapping)

```python
intervals.sort(key=lambda x: x[1])  # Q3: sort by end time
count = 0
last_end = float('-inf')

for start, end in intervals:
    if start >= last_end:            # no overlap with last picked
        count += 1
        last_end = end

return count
```

**Example — max meetings from `[(1,3),(2,4),(3,5)]`:**
```
Q1: pick earliest ending → Template 1
Q2: always valid — earliest end = most room left
Q3: sort by end time

(1,3): pick → last_end=3, count=1
(2,4): 2<3 → skip
(3,5): 3>=3 → pick → last_end=5, count=2
return 2 ✓
```
**Complexity:** O(n log n) time. O(1) space.

---

### Template 2 — Jump Game (Max Reach)

```python
max_reach = 0

for i in range(len(nums)):
    if i > max_reach:
        return False                # can't reach this position
    max_reach = max(max_reach, i + nums[i])

return True
```

**Example — can we reach end of `[2,3,1,1,4]`?**
```
Q1: extend max reach greedily → Template 2
Q2: always valid — extending reach never hurts

i=0: max_reach=2
i=1: max_reach=4
i=4: max_reach=8 → True ✓
```
**Complexity:** O(n) time. O(1) space.

---
---

# 13. Intervals

### Mental Trigger
> *"Does the problem involve ranges with start and end points — and do I need to merge, check overlap, or count resources?"*

**Why this works:** sorting by start time makes overlapping intervals adjacent. One linear scan after sorting handles most interval problems.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "merge overlapping intervals" | Sort by start, merge | 1 |
| "insert interval into sorted list" | Find overlap, merge neighbors | 1 variant |
| "minimum rooms/workers needed" | Sort + min heap of end times | 2 |
| "can attend all meetings" | Sort by start, check consecutive | 1 variant |
| "find free time" | Merge events, find gaps | 1 variant |

---

### Questions to Ask Before Coding

**Q1 — Am I merging intervals, or counting how many overlap at once?**
- Merging → Template 1
- Counting max concurrent (rooms/workers needed) → Template 2

**Q2 — What does overlap mean here?**
- Two intervals overlap if `start <= other_end`
- Touch but don't overlap: `[1,3]` and `[3,5]` — check if problem treats these as overlapping

**Q3 — Do I sort by start or end time?**
- Merging/inserting → sort by start time → Template 1
- Counting rooms → sort by start, heap tracks end times → Template 2

---

### Template 1 — Merge Intervals

```python
intervals.sort(key=lambda x: x[0])  # Q3: sort by start
merged = [intervals[0]]

for start, end in intervals[1:]:
    if start <= merged[-1][1]:       # Q2: overlaps with last merged
        merged[-1][1] = max(merged[-1][1], end)  # extend
    else:
        merged.append([start, end])  # no overlap — add new

return merged
```

**Example — merge `[[1,3],[2,6],[8,10]]`:**
```
Q1: merging → Template 1
Q2: overlap if start <= prev end
Q3: sort by start

[1,3] → start
[2,6]: 2<=3 → merge → [1,6]
[8,10]: 8>6 → add → [[1,6],[8,10]] ✓
```
**Complexity:** O(n log n) time. O(n) space.

---

### Template 2 — Minimum Meeting Rooms

```python
import heapq

intervals.sort(key=lambda x: x[0])  # Q3: sort by start
heap = []                            # tracks end times of active meetings

for start, end in intervals:
    if heap and heap[0] <= start:    # earliest-ending room is free
        heapq.heapreplace(heap, end) # reuse it
    else:
        heapq.heappush(heap, end)    # need new room

return len(heap)                     # rooms = peak concurrent meetings
```

**Example — minimum rooms for `[[1,4],[2,5],[7,9]]`:**
```
Q1: count max concurrent → Template 2
Q3: sort by start, heap tracks end times

[1,4]: push → heap=[4]
[2,5]: 2<4 → new room → heap=[4,5]
[7,9]: 7>4 → reuse → heap=[5,9]
return 2 ✓
```
**Complexity:** O(n log n) time. O(n) space.

---
---

# 14. Bit Manipulation

### Mental Trigger
> *"Can I solve this using binary representations — XOR, AND, OR, or shifts?"*

**Why this works:** bit operations run in O(1) and operate on all 32 bits simultaneously. Faster and more space-efficient than alternatives for parity, flags, and unique element problems.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "find the one unique number, all others appear twice" | XOR all | 1 |
| "count how many bits are set" | n & (n-1) loop | 2 |
| "set/check/remove a permission or flag" | Bit masking | 3 |
| "check if power of 2" | n & (n-1) == 0 | 3 |
| "multiply/divide by 2" | Left/right shift | 3 |

---

### Questions to Ask Before Coding

**Q1 — Are pairs of identical values cancelling out to leave a unique value?**
- Yes → XOR → Template 1
- No → continue to Q2

**Q2 — Am I counting, setting, checking, or clearing individual bits?**
- Counting set bits → Template 2
- Set/check/clear a specific bit → Template 3

**Q3 — Do I know which bit position I'm targeting?**
- Yes → use `1 << k` as a mask → Template 3
- No → loop through all bits or use `n & (n-1)` → Template 2

---

### Template 1 — XOR (Find Unique Element)

```python
result = 0
for num in nums:
    result ^= num               # pairs cancel (x^x=0), unique survives
return result
```

**Example — find unique in `[2,3,2,4,4]`:**
```
Q1: pairs cancel → XOR → Template 1

0^2=2, 2^3=1, 1^2=3, 3^4=7, 7^4=3 ✓
```
**Complexity:** O(n) time. O(1) space.

---

### Template 2 — Count Set Bits

```python
count = 0
while n:
    n &= (n - 1)                # clears lowest set bit each time
    count += 1
return count
```

**Example — count set bits in 13 (binary: 1101):**
```
Q2: counting set bits → Template 2

1101 & 1100 = 1100 → count=1
1100 & 1011 = 1000 → count=2
1000 & 0111 = 0000 → count=3 ✓
```
**Complexity:** O(k) where k = number of set bits. O(1) space.

---

### Template 3 — Bit Masking

```python
# set bit k
flags |= (1 << k)

# check bit k — is it on?
is_set = (flags >> k) & 1

# clear bit k
flags &= ~(1 << k)

# check if power of 2 — exactly one bit set
is_power_of_2 = n > 0 and (n & (n - 1)) == 0
```

**Example — set permission 2, check it, then remove it:**
```
Q2: set/check/clear → Template 3
Q3: target bit k=2

set:    flags |= (1<<2) → flags=4  (100)
check:  (4>>2) & 1 = 1  ✓ (set)
clear:  flags &= ~(1<<2) → flags=0
```
**Complexity:** O(1) for all operations.

---
---

# Python Mechanics

Things to know cold. These come up in almost every problem.

---

### Dict and Counter

```python
freq = {'a': 3, 'b': 1}
for k, v in freq.items():     # k=key, v=value
    print(k, v)
for k in freq:                # keys only — common mistake

sorted(freq.items(), key=lambda x: x[1], reverse=True)   # descending
sorted(freq.items(), key=lambda x: x[1], reverse=False)  # ascending

from collections import Counter
freq = Counter("aabbcc")      # {'a':2,'b':2,'c':2}
freq = Counter([1,2,2,3])     # {2:2,1:1,3:1}

('a', 3)    # tuple — what sorted() gives back
{'a': 3}    # dict — what Counter/freq looks like
print(type(x))
```

---

### Deque

```python
from collections import deque
q = deque([1, 2, 3])
q.append(4)          # add right — O(1)
q.appendleft(0)      # add left — O(1)
q.pop()              # remove right — O(1)
q.popleft()          # remove left — O(1) ← use for BFS, not list.pop(0)
```

---

### Heap

```python
import heapq
heap = []
heapq.heappush(heap, 3)
heapq.heappop(heap)          # always returns min

# max heap — negate values
heapq.heappush(heap, -5)
-heapq.heappop(heap)         # returns 5

arr = [3,1,4,1,5]
heapq.heapify(arr)           # O(n)
```

---

### Useful Built-ins

```python
for i, x in enumerate(arr):    # index + value
    print(i, x)

for a, b in zip(list1, list2): # pair two lists
    print(a, b)

arr.sort(key=lambda x: x[1])
sorted(arr, key=lambda x: -x[1])

s.split()
' '.join(words)
s[::-1]
ord('a')    # 97
chr(97)     # 'a'
```

---
---

# Space & Time Tradeoffs

> At Google and Meta, stating complexity is not enough. Reason about *why* the tradeoff is worth it.

---

### How to Articulate It

Don't say: *"This is O(n) time and O(n) space."*

Say: *"I'm using a hash map — O(n) extra space — but it eliminates the inner loop and brings time from O(n²) to O(n). Given memory isn't constrained here, that's the right tradeoff."*

Three parts:
1. What extra space you're using
2. What time improvement it buys
3. Why that tradeoff is justified

---

### When Space IS Constrained

| Normally | Space-constrained alternative |
|---|---|
| Hash set for duplicates | Sort first, check adjacent elements |
| Prefix sum array | Recompute ranges with O(1) space |
| Memoization table | Bottom-up DP with rolling array |

---
---

# Complexity Cheat Sheet

| Operation | Time |
|---|---|
| Hash map / set lookup, insert, delete | O(1) avg |
| Array access by index | O(1) |
| Array search (unsorted) | O(n) |
| Binary search | O(log n) |
| Sorting | O(n log n) |
| Single loop | O(n) |
| Nested loops | O(n²) |
| DFS / BFS on graph | O(V + E) |
| Heap push / pop | O(log n) |
| Heap build from array | O(n) |
| DP (most problems) | O(n²) |
| Backtracking (worst case) | O(2^n) or O(n!) |

---

| Structure | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) |
| Hash map | O(1) | O(1) | O(1) | O(1) |
| Stack / Queue | O(n) | O(n) | O(1) | O(1) |
| Binary search tree | O(log n) | O(log n) | O(log n) | O(log n) |
| Heap | O(1) min/max | O(n) | O(log n) | O(log n) |
| Linked list | O(n) | O(n) | O(1) | O(1) |
