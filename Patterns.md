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

**Example — are all brackets matched in `"({[]})"`?**
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
| "reverse without extra memory" | Pointer reversal | 1 |
| "detect loop or cycle" | Fast/slow pointers | 2 |
| "find middle without knowing length" | Fast/slow pointers | 2 |
| "Kth from end without full scan" | Two pointers gap k apart | 3 |
| "merge two ordered streams" | Dummy node + two pointers | 4 |

---

### Questions to Ask Before Coding

**Q1 — Do I need to track the node before the one I'm changing?**
- Yes → need `prev` pointer → Template 1
- No → continue to Q2

**Q2 — Do I need two pointers at different speeds, or a fixed gap?**
- Different speeds → Template 2 (middle, cycle detection)
- Fixed gap k → Template 3 (kth from end)

**Q3 — Does the head itself change?**
- Yes → Template 4 (dummy node as fixed anchor)
- No → manipulate directly

---

### Template 1 — Reverse (Pointer Reversal)

```python
prev, curr = None, head

while curr:
    next_node = curr.next           # save rest before losing it
    curr.next = prev                # redirect pointer backwards
    prev = curr                     # slide forward
    curr = next_node

return prev                         # prev is the new head
```

**Example — reverse `1→2→3→4`:**
```
Q1: need prev to redirect → Template 1

INITIAL: prev=None, curr=1→2→3→4
STEP 1:  next=2→3→4, 1→None | prev=1, curr=2
STEP 2:  next=3→4,   2→1    | prev=2, curr=3
STEP 3:  next=4,     3→2    | prev=3, curr=4
STEP 4:  next=None,  4→3    | prev=4, curr=None
return 4→3→2→1 ✓
```
**Complexity:** O(n) time. O(1) space.

---

### Template 2 — Fast/Slow Pointers

```python
slow, fast = head, head

while fast and fast.next:
    slow = slow.next                # 1 step
    fast = fast.next.next           # 2 steps

return slow                         # middle — or check slow==fast for cycle
```

**Example — find middle of `1→2→3→4→5`:**
```
Q1: no prev needed
Q2: different speeds → Template 2

STEP 1: slow=2, fast=3
STEP 2: slow=3, fast=5
STEP 3: fast.next=None → stop
slow=3 ← middle ✓
```
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
Q1: no prev needed
Q2: fixed gap k=2 → Template 3

After gap: left=1, right=3
STEP 1: left=2, right=4
STEP 2: left=3, right=5
STEP 3: right=None → stop
return 4 ✓
```
**Complexity:** O(n) time. O(1) space.

---

### Template 4 — Dummy Node

```python
dummy = ListNode(0)                 # fixed anchor before real list
dummy.next = head
curr = dummy

# manipulate via curr...

return dummy.next                   # real head even if it changed
```

**Example — use when merging lists or deleting head node:**
```
Q3: head might change → Template 4
Dummy absorbs head changes — always return dummy.next
```
**Complexity:** O(n) time. O(1) space.

---
---

# 7. Trees

### Mental Trigger
> *"Is the data hierarchical with branching parent-child relationships — and does the same logic apply at every level?"*

**Why this works:** every node is the root of its own subtree. Solve for one node assuming children are already solved — recursion handles the rest.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "total", "sum", "aggregate across all" | DFS bottom-up | 1 |
| "longest", "deepest", "height", "diameter" | DFS with return values | 1 |
| "validate", "compare two trees", "mirror" | DFS recursive | 1 |
| "level by level", "same depth", "layer" | BFS | 2 |
| "shortest path", "fewest steps" | BFS | 2 |
| "path sum", "root to leaf", "path between nodes" | DFS with path tracking | 3 |

---

### Questions to Ask Before Coding

**Q1 — Do I need information from the leaves to answer the question at the root?**
- Yes, and I'm combining values on the way back up → DFS → Template 1
- Yes, and I need to track the running path itself → DFS with path → Template 3
- No → continue to Q2

**Q2 — Does the answer involve processing nodes level by level?**
- Yes → BFS → Template 2

**Q3 — What do I return from each DFS call?**
- This fills your `return` statement in Template 1
- Height → `return 1 + max(left, right)`
- Sum → `return node.val + left + right`
- Valid → `return left and right and <condition>`

---

### Template 1 — DFS Recursive

```python
def dfs(node):
    if not node:                    # base case — past a leaf
        return 0                    # neutral value for this problem

    left  = dfs(node.left)          # answer from left subtree
    right = dfs(node.right)         # answer from right subtree

    return ...                      # Q3: combine left, right, node.val
```

**Example — height of tree:**
```
Q1: need leaf info, combining on way back → Template 1
Q3: return 1 + max(left, right)

        1
       / \
      2   3
     / \
    4   5

dfs(4)=1, dfs(5)=1
dfs(2)=1+max(1,1)=2
dfs(3)=1
dfs(1)=1+max(2,1)=3 ✓
```
**Complexity:** O(n) time. O(h) space where h=height.

---

### Template 2 — BFS Level by Level

```python
from collections import deque

queue = deque([root])

while queue:
    node = queue.popleft()          # process oldest node first

    # process node here

    if node.left:  queue.append(node.left)
    if node.right: queue.append(node.right)
```

**Example — level by level traversal:**
```
Q1: no leaf info needed
Q2: level by level → BFS → Template 2

queue=[1] → pop 1, add 2,3 → queue=[2,3]
pop 2, add 4,5 → queue=[3,4,5]
pop 3 → queue=[4,5]
pop 4, pop 5 → queue=[]
order: 1,2,3,4,5 ✓
```
**Complexity:** O(n) time. O(n) space.

---

### Template 3 — DFS with Path Tracking

```python
def dfs(node, current_path, current_sum):
    if not node:
        return

    current_path.append(node.val)   # add to path
    current_sum += node.val

    if not node.left and not node.right:  # leaf node
        if <condition on current_sum or path>:
            result.append(current_path[:])  # found valid path

    dfs(node.left,  current_path, current_sum)
    dfs(node.right, current_path, current_sum)

    current_path.pop()              # undo — backtrack

result = []
dfs(root, [], 0)
return result
```

**Example — find all root-to-leaf paths that sum to target:**
```
Q1: need to track running path → Template 3

        1
       / \
      2   3
     / \
    4   5

dfs(1, [], 0) → path=[1], sum=1
dfs(2, [1], 1) → path=[1,2], sum=3
dfs(4, [1,2], 3) → leaf, sum=7 → check
dfs(5, [1,2], 3) → leaf, sum=8 → check
dfs(3, [1], 1) → leaf, sum=4 → check
```
**Complexity:** O(n) time. O(h) space where h=height.

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
> *"Do I need to explore all possible combinations or paths — and can I prune dead ends early?"*

**Why this works:** build incrementally. At each step, make a choice, recurse, then undo (backtrack). Cut off paths the moment they can't lead to a valid answer.

---

### Translation Table

| You hear this... | Pattern | Template |
|---|---|---|
| "all subsets", "power set" | Include/exclude each element | 1 |
| "all permutations" | Try each unused element at each position | 2 |
| "all combinations that sum to target" | Running sum with pruning | 3 |
| "word search", "path in grid", "fill board" | Grid backtracking | 4 |
| "solve puzzle", "fill grid" | Try, recurse, undo on contradiction | 3 variant |

---

### Questions to Ask Before Coding

**Q1 — Does order matter in the output?**
- No (subsets) → iterate forward from `start`, never revisit → Template 1
- Yes (permutations) → try all unused elements at each position → Template 2

**Q2 — Can elements be reused?**
- No → pass `i + 1` as next start → Template 1
- Yes → pass `i` as next start (same element allowed again) → Template 3

**Q3 — Am I searching through a 2D grid?**
- Yes → grid backtracking → Template 4

**Q4 — What is the pruning condition?**
- This goes in your early `return` before the loop
- Sum exceeds target → `if remaining < 0: return`
- Out of bounds or visited → `if not valid: return`
- Invalid board state → `if not valid(board): return`

---

### Template 1 — Subsets (No reuse, order doesn't matter)

```python
def backtrack(start, current):
    result.append(current[:])       # every state is valid — add it

    for i in range(start, len(nums)):
        current.append(nums[i])     # choose
        backtrack(i + 1, current)   # recurse — i+1 means no reuse
        current.pop()               # undo

result = []
backtrack(0, [])
return result
```

**Example — all subsets of `[1,2,3]`:**
```
Q1: order doesn't matter → Template 1
Q2: no reuse → i+1
Q3: no pruning needed

[] → add []
include 1 → [1] → add [1]
  include 2 → [1,2] → add [1,2]
    include 3 → [1,2,3] → add [1,2,3]
  undo 2, include 3 → [1,3] → add [1,3]
undo 1, include 2 → [2] → ...
```
**Complexity:** O(2^n) time. O(n) space.

---

### Template 2 — Permutations (Order matters)

```python
def backtrack(current):
    if len(current) == len(nums):
        result.append(current[:])   # complete permutation
        return

    for num in nums:
        if num in current:          # skip already used
            continue
        current.append(num)
        backtrack(current)
        current.pop()

result = []
backtrack([])
return result
```

**Example — all permutations of `[1,2,3]`:**
```
Q1: order matters → Template 2
Q2: no reuse — skip if already in current

[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1] ✓
```
**Complexity:** O(n!) time. O(n) space.

---

### Template 3 — Combination Sum (Reuse allowed, pruning)

```python
def backtrack(start, current, remaining):
    if remaining == 0:
        result.append(current[:])   # valid combination found
        return
    if remaining < 0:
        return                      # pruned — exceeded target

    for i in range(start, len(candidates)):
        current.append(candidates[i])
        backtrack(i, current, remaining - candidates[i])  # i not i+1 — allows reuse
        current.pop()

result = []
backtrack(0, [], target)
return result
```

**Example — combinations summing to 7 from `[2,3,6,7]`:**
```
Q1: order doesn't matter → forward iteration
Q2: reuse allowed → pass i not i+1
Q4: prune when remaining < 0

try 2 → rem=5 → try 2 → rem=3 → try 3 → rem=0 → add [2,2,3]
try 7 → rem=0 → add [7]
result: [[2,2,3],[7]] ✓
```
**Complexity:** O(2^n) worst case, pruning reduces in practice.

---

### Template 4 — Grid Backtracking (Word Search)

```python
def backtrack(r, c, index):
    if index == len(word):
        return True                 # found entire word

    if r < 0 or r >= rows or c < 0 or c >= cols:
        return False                # out of bounds
    if grid[r][c] != word[index]:
        return False                # wrong character
    if (r, c) in visited:
        return False                # already used this cell

    visited.add((r, c))            # mark as used

    found = (backtrack(r+1, c, index+1) or   # explore all 4 directions
             backtrack(r-1, c, index+1) or
             backtrack(r, c+1, index+1) or
             backtrack(r, c-1, index+1))

    visited.remove((r, c))         # undo — unmark for other paths
    return found

visited = set()
for r in range(rows):
    for c in range(cols):
        if backtrack(r, c, 0):
            return True
return False
```

**Example — find "ABC" in grid `[["A","B","C"],["D","E","F"]]`:**
```
Q3: searching 2D grid → Template 4
Q4: prune if out of bounds, wrong char, or already visited

start at (0,0)='A' → match index=0
  move right (0,1)='B' → match index=1
    move right (0,2)='C' → match index=2 → found ✓
```
**Complexity:** O(rows * cols * 4^len(word)) time. O(len(word)) space.

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
