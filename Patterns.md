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

### Questions to Ask Before Coding

**Q1 — Do I need existence only, or do I need to store something with each element?**
- Existence only → `set`
- Need index, count, or group → `dict`

**Q2 — What is the key? What is the value?**
- This fills your `seen[key] = value` directly
- element → index (two sum)
- element → count (frequency)
- sorted signature → list of words (grouping)

**Q3 — One pass or two?**
- Check and store simultaneously → one pass (complement trick)
- Build full map first, then query → two pass (frequency comparison)

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

### Questions to Ask Before Coding

**Q1 — Is the input sorted, or can I sort it without breaking the problem?**
- Yes → two pointers inward is valid (Template 1, Template 3)
- No, and order matters → same direction slow/fast pointers (Template 2)

**Q2 — Do the pointers move toward each other, or in the same direction?**
- Toward each other → comparing elements from both ends (sum, palindrome, water)
- Same direction → one reads, one writes (in-place removal)

**Q3 — What is the move condition?**
- This fills your `if/elif/else` inside the loop
- Sum too small → move left up
- Sum too big → move right down
- Element invalid → skip read, don't advance write

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

### Questions to Ask Before Coding

**Q1 — Fixed or variable window size?**
- Size given (e.g. k=3) → **Template 1**
- Find min/max length satisfying a condition → **Template 2**

**Q2 — What makes the window invalid?**
- This is your `while` shrink condition in Template 2
- "no duplicates" → `while arr[right] in state`
- "sum exceeds limit" → `while current_sum > limit`

**Q3 — What state do I track inside the window?**
- Running total → plain variable
- Membership → `set`
- Frequency → `dict`

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

```
Input:    "({[]})"
Question: Are all brackets properly opened and closed in the right order?
Output:   True

Without stack:
- No way to remember which bracket was opened most recently

With stack:
- See ( → push → stack=[(]
- See { → push → stack=[(,{]
- See [ → push → stack=[(,{,[]
- See ] → matches [ → pop → stack=[(,{]
- See } → matches { → pop → stack=[(]
- See ) → matches ( → pop → stack=[]
- Stack empty → True
```

**Why this works:** last opened = first to close. Stack is LIFO — perfect match.

---

### When You'll See This In Interviews

**Example 1 — Code editor**
*"Validate whether all opening tags in an HTML document have matching closing tags."*

```
Input:    "<div><p></p></div>"
Question: Are all tags properly matched?
Output:   True

- see <div> → push → stack=[div]
- see <p>   → push → stack=[div,p]
- see </p>  → matches p → pop → stack=[div]
- see </div>→ matches div → pop → stack=[]
- stack empty → True ✓
```

---

**Example 2 — Stock prices**
*"For each day's stock price, find how many consecutive previous days had a lower price."*

```
Input:    [2, 1, 3, 4]
Question: For each price, how many days back was the last higher price?
Output:   [0, 0, 2, 3]

- price=2 → stack=[0]
- price=1 → 1<2, push → stack=[0,1]
- price=3 → 3>1 → pop. 3>2 → pop. push → stack=[2] → result[0]=3, result[1]=3
- price=4 → 4>3 → pop. push → stack=[3] → result[2]=4
```

→ Monotonic stack. Pop when current breaks the order.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "matching brackets", "valid nesting" | Stack push/pop | LIFO matches opening to most recent closing |
| "undo", "go back", "reverse order" | Stack | Last action is first to undo |
| "next greater element", "daily temperatures" | Monotonic stack | Pop when current element breaks the order |
| "evaluate expression", "calculator" | Stack | Push operands, evaluate on operator |
| "nested structure", "recursive-looking" | Stack | Simulate recursion without recursion |

---

### Template 1 — Matching / Validation

```
Input:    "({[]})"
Question: Are all brackets valid and matched?
Output:   True
```

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
- see ( → push → stack=[(]
- see { → push → stack=[(,{]
- see [ → push → stack=[(,{,[]
- see ] → matches [ → pop → stack=[(,{]
- see } → matches { → pop → stack=[(]
- see ) → matches ( → pop → stack=[]
- empty → True ✓
```

---

### Template 2 — Monotonic Stack

```
Input:    [2, 1, 3, 4]
Question: For each element, find the next greater element
Output:   [3, 3, 4, -1]
```

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
- i=0(2) → stack=[0]
- i=1(1) → 1<2 → push → stack=[0,1]
- i=2(3) → 3>1 → result[1]=3, pop. 3>2 → result[0]=3, pop. push → stack=[2]
- i=3(4) → 4>3 → result[2]=4, pop. push → stack=[3]
result=[3,3,4,-1] ✓
```

**Complexity:** O(n) time, O(n) space

---
---

# 5. Binary Search

### Mental Trigger
> *"Is the input sorted — or can I define a condition where everything to the left fails and everything to the right passes?"*

```
Input:    [1, 3, 5, 7, 9], target=7
Question: What index is the target at?
Output:   3

Brute force:
- Scan every element → O(n)

Binary search:
- mid=5 → 5<7 → search right half
- mid=7 → found → return index 3
- O(log n)
```

**Why this works:** each step eliminates half the remaining candidates.

---

### When You'll See This In Interviews

**Example 1 — Server logs**
*"Find the first error that occurred after a given timestamp in sorted logs."*

```
Input:    [100, 200, 300, 400, 500], target=250
Question: What is the first timestamp after 250?
Output:   300

- mid=300 → 300>250 → search left half
- mid=200 → 200<250 → search right half
- mid=300 → first value > 250 → return 300
```

→ Binary search for leftmost value satisfying condition.

---

**Example 2 — Find broken version**
*"One version introduced a bug. All versions after it are also broken. Find the first broken one."*

```
Input:    [good, good, good, broken, broken], first broken=4
Question: What is the first broken version?
Output:   4

- mid=3(good) → broken must be right → search right
- mid=4(broken) → could be first → record, search left
- mid=4 → confirmed → return 4
```

→ Binary search on condition. Once broken, stays broken.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "sorted array", "find target" | Classic binary search | Eliminate half each step |
| "first/last occurrence" | Binary search with boundary tracking | Don't stop at first match |
| "minimum/maximum that satisfies condition" | Binary search on answer | Check if mid works, search accordingly |
| "find broken version", "first bad commit" | Binary search on condition | Monotonic — once broken, stays broken |
| "rotated sorted array" | Modified binary search | One half is always sorted |

---

### Template 1 — Classic Binary Search

```
Input:    [1, 3, 5, 7, 9], target=7
Question: What index is the target at?
Output:   3
```

```python
left, right = 0, len(arr) - 1

while left <= right:
    mid = (left + right) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return -1
```

```
- left=0, right=4, mid=2(5) → 5<7 → left=3
- left=3, right=4, mid=3(7) → 7==7 → return 3 ✓
```

---

### Template 2 — Binary Search on Answer

```
Input:    weights=[1,2,3,4,5], days=3
Question: What is the minimum ship capacity to deliver all packages in 3 days?
Output:   6
```

```python
left, right = max(weights), sum(weights)

while left < right:
    mid = (left + right) // 2
    if condition(mid):
        right = mid
    else:
        left = mid + 1

return left
```

```
- left=5, right=15, mid=10 → works in 2 days → right=10
- left=5, right=10, mid=7  → works in 3 days → right=7
- left=5, right=7,  mid=6  → works in 3 days → right=6
- left=5, right=6,  mid=5  → needs 4 days → left=6
- left=right=6 → return 6 ✓
```

---

### Template 3 — First/Last Occurrence

```
Input:    [1, 2, 2, 2, 3], target=2
Question: What is the first index where target appears?
Output:   1
```

```python
left, right = 0, len(arr) - 1
result = -1

while left <= right:
    mid = (left + right) // 2
    if arr[mid] == target:
        result = mid
        right = mid - 1    # keep searching left for first occurrence
    elif arr[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return result
```

```
- mid=2(index 2) → match → result=2, search left
- mid=2(index 1) → match → result=1, search left
- mid=2(index 0) → no match → search right
- done → return 1 ✓
```

**Complexity:** O(log n) time, O(1) space

---
---

# 6. Linked List

### Mental Trigger
> *"Is the data sequential where each item only knows what comes next — and do I need to manipulate connections rather than values?"*

```
Input:    1 → 2 → 3 → 4 → None
Question: Reverse the linked list
Output:   4 → 3 → 2 → 1 → None

Without pointer manipulation:
- Need extra memory to store and rebuild

With pointer reversal:
- Redirect each node's next pointer backwards
- O(1) space
```

**Why this works:** you don't copy — you redirect.

---

### When You'll See This In Interviews

**Example 1 — Detect loop**
*"Detect if a payment processor is stuck in a loop processing the same transactions."*

```
Input:    1 → 2 → 3 → 4 → 2 (cycle back to 2)
Question: Does a cycle exist?
Output:   True

- slow=1, fast=1
- slow=2, fast=3
- slow=3, fast=2  (fast looped back)
- slow=4, fast=4  → they meet → cycle exists ✓
```

→ Fast/slow pointers. They meet only if a cycle exists.

---

**Example 2 — Kth from end**
*"Find the Kth most recent server error without knowing total count."*

```
Input:    1 → 2 → 3 → 4 → 5, k=2
Question: What is the 2nd node from the end?
Output:   4

- move right k=2 steps ahead → left=1, right=3
- move both until right hits None
- left=2, right=4
- left=3, right=5
- left=4, right=None → stop → return 4 ✓
```

→ Two pointers gap k apart. When right hits end, left is k from end.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "reverse without extra memory" | Pointer reversal | Redirect pointers in place |
| "detect loop or cycle" | Fast/slow pointers | They meet only if a cycle exists |
| "find middle without knowing length" | Fast/slow pointers | Fast reaches end when slow is halfway |
| "Kth from end without full scan" | Two pointers gap k apart | Front hits end, back is k behind |
| "merge two ordered streams" | Dummy node + two pointers | Pick smaller node each step |

---

### Template 1 — Reverse a Linked List

```
Input:    1 → 2 → 3 → 4 → None
Question: Reverse the list in-place
Output:   4 → 3 → 2 → 1 → None
```

```python
prev, curr = None, head
while curr:
    next_node = curr.next    # save rest before redirecting
    curr.next = prev         # reverse the pointer
    prev = curr
    curr = next_node
return prev
```

```
- prev=None, curr=1
- save next=2, point 1→None → prev=1, curr=2
- save next=3, point 2→1    → prev=2, curr=3
- save next=4, point 3→2    → prev=3, curr=4
- save next=None, point 4→3 → prev=4, curr=None
return 4 → 3 → 2 → 1 → None ✓
```

---

### Template 2 — Fast/Slow Pointers

```
Input:    1 → 2 → 3 → 4 → 5
Question: Find the middle node
Output:   3
```

```python
slow, fast = head, head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
return slow
```

```
- slow=1, fast=1
- slow=2, fast=3
- slow=3, fast=5
- fast.next=None → stop → return 3 ✓
```

---

### Template 3 — Kth From End

```
Input:    1 → 2 → 3 → 4 → 5, k=2
Question: Find the 2nd node from the end
Output:   4
```

```python
left, right = head, head
for _ in range(k):
    right = right.next

while right:
    left = left.next
    right = right.next

return left
```

```
- move right k=2 ahead → left=1, right=3
- left=2, right=4
- left=3, right=5
- left=4, right=None → stop → return 4 ✓
```

**Complexity:** O(n) time, O(1) space

---
---

# 7. Trees

### Mental Trigger
> *"Is the data hierarchical with branching parent-child relationships — and does the same logic apply at every level?"*

```
Input:        1
             / \
            2   3
           / \
          4   5

Question: What is the height of this tree?
Output:   3

DFS thinking:
- Can't know height of 1 until I know height of 2 and 3
- Can't know height of 2 until I know height of 4 and 5
- 4 and 5 are leaves → height=1
- 2: 1 + max(1,1) = 2
- 1: 1 + max(2,1) = 3
```

**Why this works:** every node is the root of its own subtree. Solve for small, combine upward.

---

### When You'll See This In Interviews

**Example 1 — File system**
*"Calculate total storage used by a folder including all nested subfolders."*

```
Input:    root(10) → folder1(20) → file1(5)
                   → folder2(30)
Question: What is total storage from root?
Output:   65

- file1 → 5
- folder1 → 20 + 5 = 25
- folder2 → 30
- root → 10 + 25 + 30 = 65
```

→ DFS bottom-up. Leaves return first, parents combine.

---

**Example 2 — Notification system**
*"Send notification to all employees at the same level simultaneously."*

```
Input:    CEO → VP1, VP2
               VP1 → Mgr1, Mgr2
Question: Process level by level
Output:   [CEO], [VP1,VP2], [Mgr1,Mgr2]

- queue=[CEO]
- pop CEO → add VP1,VP2 → queue=[VP1,VP2]
- pop VP1 → add Mgr1,Mgr2. pop VP2 → queue=[Mgr1,Mgr2]
```

→ BFS. Queue processes level by level naturally.

---

### DFS vs BFS

**Use DFS when** you need information from leaves to answer at root. Go deep, combine on way back.

**Use BFS when** the answer is about levels or shortest path. Queue processes all nodes at depth 1 before depth 2.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "total", "sum", "aggregate across all" | DFS bottom-up | Need leaf values first, combine upward |
| "longest", "deepest", "height" | DFS with return values | Go deep, build answer on way back |
| "validate", "compare two trees" | DFS recursive | Same check at every node |
| "level by level", "same depth" | BFS | Queue processes level by level |
| "shortest path", "fewest steps" | BFS | First time you reach destination = shortest |

---

### Template 1 — DFS Recursive

```
Input:        1
             / \
            2   3
           / \
          4   5
Question: What is the height?
Output:   3
```

```python
def dfs(node):
    if not node:
        return 0
    left  = dfs(node.left)
    right = dfs(node.right)
    return 1 + max(left, right)
```

```
- dfs(4)=1, dfs(5)=1
- dfs(2)=1+max(1,1)=2
- dfs(3)=1
- dfs(1)=1+max(2,1)=3 ✓
```

---

### Template 2 — BFS Level by Level

```
Input:    same tree above
Question: Return nodes level by level
Output:   [[1],[2,3],[4,5]]
```

```python
from collections import deque
queue = deque([root])
while queue:
    node = queue.popleft()
    if node.left: queue.append(node.left)
    if node.right: queue.append(node.right)
```

```
- queue=[1] → pop 1, add 2,3 → queue=[2,3]
- pop 2, add 4,5 → queue=[3,4,5]
- pop 3, no kids → queue=[4,5]
- pop 4, pop 5 → queue=[]
order: 1,2,3,4,5 ✓
```

**Complexity:** O(n) time. O(h) space DFS. O(n) space BFS.

---
---

# 8. Heap / Priority Queue

### Mental Trigger
> *"Do I need to repeatedly find the smallest or largest element from a changing collection?"*

```
Input:    [3, 1, 4, 1, 5, 9, 2, 6], k=3
Question: What are the 3 largest elements?
Output:   [5, 6, 9]

Brute force:
- Sort entire array → O(n log n)

Min heap of size k:
- Maintain heap of 3 largest seen so far
- New element beats minimum in heap? Replace it
- O(n log k)
```

**Why this works:** you don't need to sort everything. Just maintain the top k.

---

### When You'll See This In Interviews

**Example 1 — Live leaderboard**
*"Show the top 10 scores at any moment as new scores come in."*

```
Input:    scores=[5,3,8,1,9,2,7,4,6,10,11], k=3
Question: What are the top 3 scores after all arrive?
Output:   [9,10,11]

- heap=[5,3,8] → size=3
- 1 < min(3) → skip
- 9 > min(3) → replace → heap=[5,8,9]
- 2 < min(5) → skip
- 7 > min(5) → replace → heap=[7,8,9]
- ...continue
- final heap=[9,10,11] ✓
```

---

**Example 2 — Merge sorted feeds**
*"Merge K sorted news feeds into one sorted feed."*

```
Input:    [[1,4,7],[2,5,8],[3,6,9]]
Question: Merge into one sorted list
Output:   [1,2,3,4,5,6,7,8,9]

- push first of each → heap=[(1,0,0),(2,1,0),(3,2,0)]
- pop 1 → output 1, push next from list 0 → heap=[(2,1,0),(3,2,0),(4,0,1)]
- pop 2 → output 2, push next from list 1...
- always pop global minimum → sorted output
```

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "top K", "K largest", "K most frequent" | Min heap size K | Eject smallest when new element wins |
| "K smallest" | Max heap size K | Eject largest when new element wins |
| "running median" | Two heaps (max + min) | Balance halves, median at boundary |
| "merge K sorted lists" | Min heap with K elements | Always pop global minimum next |
| "next task by priority" | Min/max heap | O(1) access to highest priority |

---

### Template 1 — Top K Largest

```
Input:    [1,1,1,2,2,3], k=2
Question: What are the top 2 most frequent elements?
Output:   [1, 2]
```

```python
import heapq
heap = []
for num in nums:
    heapq.heappush(heap, num)
    if len(heap) > k:
        heapq.heappop(heap)
return heap
```

```
- push 3 → [3]
- push 1 → [1,3]
- push 4 → [1,3,4] → size=3
- push 1 → [1,1,3,4] → pop 1 → [1,3,4]
- push 5 → pop 1 → [3,4,5]
- push 9 → pop 3 → [4,5,9]
- push 2 → pop 2 → [4,5,9]
- push 6 → pop 4 → [5,6,9] ✓
```

---

### Template 2 — Merge K Sorted Lists

```
Input:    [[1,4,7],[2,5,8],[3,6,9]]
Question: Merge into one sorted list
Output:   [1,2,3,4,5,6,7,8,9]
```

```python
import heapq
heap = []
for i, lst in enumerate(lists):
    if lst:
        heapq.heappush(heap, (lst[0], i, 0))

result = []
while heap:
    val, i, j = heapq.heappop(heap)
    result.append(val)
    if j + 1 < len(lists[i]):
        heapq.heappush(heap, (lists[i][j+1], i, j+1))
return result
```

```
- initial heap=[(1,0,0),(2,1,0),(3,2,0)]
- pop 1 → result=[1], push (4,0,1)
- pop 2 → result=[1,2], push (5,1,1)
- pop 3 → result=[1,2,3], push (6,2,1)
...continues until all lists exhausted ✓
```

**Complexity:** O(n log k) time, k=number of lists

---
---

# 9. Backtracking

### Mental Trigger
> *"Do I need to explore all possible combinations or paths — and can I prune dead ends early?"*

```
Input:    [1, 2, 3]
Question: Generate all possible subsets
Output:   [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]

Brute force:
- Try every possible combination blindly

Backtracking:
- At each element, try including it and excluding it
- Build incrementally, undo when done exploring that path
```

---

### When You'll See This In Interviews

**Example 1 — Permission combinations**
*"Generate all possible permission sets a user can have."*

```
Input:    permissions=[read, write, execute]
Question: What are all possible permission combinations?
Output:   all 8 subsets

- include read → [read] → include write → [read,write] → ...
- undo write → [read] → include execute → [read,execute]
- undo read → [] → include write → [write] → ...
```

---

**Example 2 — Combination sum**
*"Find all combinations of numbers that sum to a target."*

```
Input:    candidates=[2,3,6,7], target=7
Question: Which combinations sum to 7?
Output:   [[2,2,3],[7]]

- try 2 → remaining=5 → try 2 → remaining=3 → try 3 → remaining=0 → found [2,2,3]
- try 7 → remaining=0 → found [7]
```

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "all subsets", "power set" | Backtrack include/exclude | Binary choice at each element |
| "all permutations" | Backtrack with used[] | Try each unused element at each position |
| "all combinations that sum to target" | Backtrack with running sum | Prune when sum exceeds target |
| "solve puzzle", "fill grid" | Backtrack with validity check | Try, recurse, undo on contradiction |
| "all paths in graph/tree" | DFS with path tracking | Explore, add to path, remove on return |

---

### Template 1 — Subsets

```
Input:    [1, 2, 3]
Question: Generate all subsets
Output:   [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

```python
def backtrack(start, current):
    result.append(current[:])
    for i in range(start, len(nums)):
        current.append(nums[i])
        backtrack(i + 1, current)
        current.pop()

result = []
backtrack(0, [])
return result
```

```
- start at [] → add []
- include 1 → [1] → add [1]
  - include 2 → [1,2] → add [1,2]
    - include 3 → [1,2,3] → add [1,2,3]
  - undo 2, include 3 → [1,3] → add [1,3]
- undo 1, include 2 → [2] → add [2]
  - include 3 → [2,3] → add [2,3]
- undo 2, include 3 → [3] → add [3]
```

---

### Template 2 — Combination Sum

```
Input:    candidates=[2,3,6,7], target=7
Question: Find all combinations that sum to target
Output:   [[2,2,3],[7]]
```

```python
def backtrack(start, current, remaining):
    if remaining == 0:
        result.append(current[:])
        return
    if remaining < 0:
        return
    for i in range(start, len(candidates)):
        current.append(candidates[i])
        backtrack(i, current, remaining - candidates[i])
        current.pop()

result = []
backtrack(0, [], target)
return result
```

```
- try 2 → remaining=5 → try 2 → remaining=3 → try 3 → remaining=0 → add [2,2,3]
- try 7 → remaining=0 → add [7]
```

**Complexity:** O(2^n) worst case

---
---

# 10. Graphs

### Mental Trigger
> *"Is the data a network of connected nodes where I need to traverse connections or find paths?"*

```
Input:    grid=[[1,1,0],
                [1,0,0],
                [0,0,1]]
Question: How many islands (connected groups of 1s) exist?
Output:   2

- Start at (0,0)=1 → DFS, mark all connected 1s as visited
- (0,0),(0,1),(1,0) all connected → island 1
- (2,2)=1, not visited → island 2
```

**Why this works:** DFS/BFS from each unvisited node explores the entire connected component.

---

### When You'll See This In Interviews

**Example 1 — Social network**
*"Find all users connected within 2 degrees of separation."*

```
Input:    A→B, B→C, A→D, target=A, degrees=2
Question: Who is within 2 degrees of A?
Output:   [B, D, C]

- level 1 (degree 1): B, D
- level 2 (degree 2): C (from B)
- stop at depth 2
```

→ BFS. Level = degree of separation.

---

**Example 2 — Build order**
*"Given package dependencies, find a valid build order."*

```
Input:    A→C, B→C, C→D
Question: What order can we build without dependency conflicts?
Output:   [A, B, C, D]

- A has no dependencies → build first
- B has no dependencies → build second
- C depends on A,B (both done) → build third
- D depends on C (done) → build last
```

→ Topological sort. Process nodes with no remaining dependencies first.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "connected components", "count islands" | DFS/BFS + visited set | Explore each component, mark visited |
| "shortest path", "fewest steps" | BFS | Level = distance, first reach = shortest |
| "can reach", "is path possible" | DFS/BFS | Explore from source, check if target found |
| "dependency order", "valid ordering" | Topological sort | Process nodes with no remaining dependencies |
| "spread", "infection", "multi-source" | Multi-source BFS | Start BFS from all sources simultaneously |

---

### Template 1 — DFS (Count Islands)

```
Input:    [[1,1,0],[1,0,0],[0,0,1]]
Question: How many islands?
Output:   2
```

```python
def dfs(r, c):
    if r < 0 or r >= rows or c < 0 or c >= cols:
        return
    if grid[r][c] != '1':
        return
    grid[r][c] = '0'
    dfs(r+1,c); dfs(r-1,c); dfs(r,c+1); dfs(r,c-1)

count = 0
for r in range(rows):
    for c in range(cols):
        if grid[r][c] == '1':
            dfs(r, c)
            count += 1
return count
```

```
- (0,0)=1 → DFS → marks (0,0),(0,1),(1,0) as 0 → count=1
- (2,2)=1 → DFS → marks (2,2) as 0 → count=2
return 2 ✓
```

---

### Template 2 — BFS Shortest Path

```
Input:    graph={A:[B,D], B:[C], D:[], C:[]}, start=A, target=C
Question: Fewest steps from A to C?
Output:   2
```

```python
from collections import deque
queue = deque([start])
visited = {start}
distance = 0

while queue:
    for _ in range(len(queue)):
        node = queue.popleft()
        if node == target:
            return distance
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    distance += 1
return -1
```

```
- queue=[A], distance=0
- pop A → not target → add B,D → queue=[B,D], distance=1
- pop B → not target → add C → queue=[D,C], distance=2
- pop D → not target → queue=[C]
- pop C → target! → return 2 ✓
```

---

### Template 3 — Topological Sort

```
Input:    A→C, B→C, C→D
Question: Valid build order?
Output:   [A, B, C, D]
```

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

return order if len(order) == len(graph) else []
```

```
- in_degree: A=0, B=0, C=2, D=1
- queue=[A,B]
- pop A → order=[A], C in_degree→1
- pop B → order=[A,B], C in_degree→0 → add C
- pop C → order=[A,B,C], D in_degree→0 → add D
- pop D → order=[A,B,C,D] ✓
```

**Complexity:** O(V+E) time

---
---

# 11. Dynamic Programming

### Mental Trigger
> *"Am I solving the same subproblem multiple times — and can I build the answer from smaller answers I've already computed?"*

```
Input:    n=5
Question: How many ways to climb 5 stairs taking 1 or 2 steps at a time?
Output:   8

Without DP:
- Recursively recompute ways(3), ways(2) multiple times

With DP:
- ways(1)=1, ways(2)=2
- ways(3)=ways(2)+ways(1)=3
- ways(4)=ways(3)+ways(2)=5
- ways(5)=ways(4)+ways(3)=8
- Each subproblem computed once
```

**Why this works:** store answers to subproblems. Never recompute.

---

### When You'll See This In Interviews

**Example 1 — Knapsack**
*"Given items with weights and values, maximize value within a weight limit."*

```
Input:    items=[(2,3),(3,4)], capacity=5
Question: Maximum value within weight limit?
Output:   7

- At capacity 2: take item1 → value=3
- At capacity 3: take item2 → value=4
- At capacity 5: take both → value=7
```

---

**Example 2 — Coin change**
*"Find minimum coins to make an amount."*

```
Input:    coins=[1,2,5], amount=6
Question: Minimum coins to make 6?
Output:   2 (5+1)

- dp[0]=0
- dp[1]=1 (one 1-coin)
- dp[2]=1 (one 2-coin)
- dp[5]=1 (one 5-coin)
- dp[6]=min(dp[5]+1, dp[4]+1, dp[1]+1)=2 ✓
```

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "number of ways", "count paths" | 1D DP | Each answer builds from previous |
| "max/min value with constraint" | Knapsack DP | Include or exclude each item |
| "shortest edit distance" | 2D DP | Build from matching substrings |
| "coin change", "unbounded items" | Unbounded knapsack | Items reusable, iterate forward |
| "palindrome substring" | 2D DP on string | Check expanding windows |

---

### Template 1 — 1D DP

```
Input:    n=5
Question: Ways to climb stairs (1 or 2 steps)
Output:   8
```

```python
dp = [0] * (n + 1)
dp[0] = 1
dp[1] = 1
for i in range(2, n + 1):
    dp[i] = dp[i-1] + dp[i-2]
return dp[n]
```

```
- dp[0]=1, dp[1]=1
- dp[2]=2, dp[3]=3, dp[4]=5, dp[5]=8 ✓
```

---

### Template 2 — 0/1 Knapsack

```
Input:    items=[(2,3),(3,4),(4,5)], capacity=5
Question: Maximum value within capacity?
Output:   7
```

```python
dp = [0] * (capacity + 1)
for weight, value in items:
    for w in range(capacity, weight - 1, -1):
        dp[w] = max(dp[w], dp[w - weight] + value)
return dp[capacity]
```

```
- after (2,3): dp[2]=3, dp[3]=3, dp[4]=3, dp[5]=3
- after (3,4): dp[3]=4, dp[5]=max(3,dp[2]+4)=7
- after (4,5): dp[5]=max(7,dp[1]+5)=7
return 7 ✓
```

---

### Template 3 — Unbounded Knapsack (Coin Change)

```
Input:    coins=[1,2,5], amount=6
Question: Minimum coins to make 6?
Output:   2
```

```python
dp = [float('inf')] * (amount + 1)
dp[0] = 0
for coin in coins:
    for w in range(coin, amount + 1):
        dp[w] = min(dp[w], dp[w - coin] + 1)
return dp[amount] if dp[amount] != float('inf') else -1
```

```
- after coin 1: dp=[0,1,2,3,4,5,6]
- after coin 2: dp=[0,1,1,2,2,3,3]
- after coin 5: dp=[0,1,1,2,2,1,2]
return dp[6]=2 ✓
```

**Complexity:** O(n²) to O(n³) depending on sub-pattern

---
---

# 12. Greedy

### Mental Trigger
> *"Can I make the locally best choice at each step without reconsidering — and does that always lead to the global best?"*

```
Input:    intervals=[(1,3),(2,4),(3,5)]
Question: Maximum non-overlapping intervals?
Output:   2

Greedy:
- Sort by end time → [(1,3),(2,4),(3,5)]
- Pick (1,3) → last_end=3
- (2,4): 2<3 → overlaps → skip
- (3,5): 3>=3 → pick → last_end=5
- count=2
```

**Why this works:** picking earliest-ending interval always leaves the most room for future intervals.

---

### When You'll See This In Interviews

**Example 1 — Meeting scheduler**
*"Find maximum meetings one person can attend."*

```
Input:    meetings=[(1,3),(2,4),(3,5),(4,6)]
Question: Maximum non-overlapping meetings?
Output:   2

- sort by end: [(1,3),(2,4),(3,5),(4,6)]
- pick (1,3) → last_end=3
- (2,4): 2<3 → skip
- (3,5): 3>=3 → pick → last_end=5
- (4,6): 4<5 → skip
count=2
```

---

**Example 2 — Jump game**
*"Given jump lengths, can you reach the last index?"*

```
Input:    [2, 3, 1, 1, 4]
Question: Can you reach the last index?
Output:   True

- i=0: max_reach=max(0,0+2)=2
- i=1: max_reach=max(2,1+3)=4
- i=2: max_reach=max(4,2+1)=4
- i=3: max_reach=max(4,3+1)=4
- i=4: max_reach=max(4,4+4)=8 → past end → True
```

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "maximum events/meetings to attend" | Sort by end time, greedy pick | Earliest end = most future room |
| "can you reach the end" | Track max reachable index | Extend greedily at each step |
| "minimum platforms/resources needed" | Sort + track overlaps | Count concurrent needs |
| "assign jobs to minimize time" | Sort by duration | Shortest job first reduces average wait |

---

### Template 1 — Interval Scheduling

```
Input:    [(1,3),(2,4),(3,5)]
Question: Maximum non-overlapping intervals?
Output:   2
```

```python
intervals.sort(key=lambda x: x[1])
count = 0
last_end = float('-inf')

for start, end in intervals:
    if start >= last_end:
        count += 1
        last_end = end
return count
```

```
- (1,3): 1>=-inf → pick, last_end=3, count=1
- (2,4): 2<3 → skip
- (3,5): 3>=3 → pick, last_end=5, count=2
return 2 ✓
```

---

### Template 2 — Jump Game

```
Input:    [2, 3, 1, 1, 4]
Question: Can you reach the last index?
Output:   True
```

```python
max_reach = 0
for i in range(len(nums)):
    if i > max_reach:
        return False
    max_reach = max(max_reach, i + nums[i])
return True
```

```
- i=0: max_reach=2
- i=1: max_reach=4
- i=2: max_reach=4
- i=3: max_reach=4
- i=4: max_reach=8 → return True ✓
```

**Complexity:** O(n log n) sorting-based, O(n) scan-based

---
---

# 13. Intervals

### Mental Trigger
> *"Does the problem involve ranges with start and end points — and do I need to merge, check overlap, or insert?"*

```
Input:    [[1,3],[2,6],[8,10]]
Question: Merge all overlapping intervals
Output:   [[1,6],[8,10]]

- [1,3] → start with [1,3]
- [2,6]: 2<=3 → overlaps → merge → [1,6]
- [8,10]: 8>6 → no overlap → add → [[1,6],[8,10]]
```

**Why this works:** sorting by start time makes overlapping intervals adjacent — then one pass merges them.

---

### When You'll See This In Interviews

**Example 1 — Calendar free time**
*"Given a person's calendar, find all free time slots."*

```
Input:    [[9,10],[11,12],[14,15]]
Question: What are the free slots in the day?
Output:   [[10,11],[12,14]]

- merge all events first
- gaps between merged events = free time
- gap after [9,10] and before [11,12] = [10,11]
- gap after [11,12] and before [14,15] = [12,14]
```

---

**Example 2 — Minimum meeting rooms**
*"Find minimum workers needed so no two tasks overlap."*

```
Input:    [[1,4],[2,5],[7,9]]
Question: Minimum workers needed?
Output:   2

- sort by start: [[1,4],[2,5],[7,9]]
- worker1 takes [1,4] → heap=[4]
- [2,5]: 2<4 → new worker → heap=[4,5]
- [7,9]: 7>4 → reuse worker1 → heap=[5,9]
max heap size = 2 ✓
```

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "merge overlapping intervals" | Sort by start, merge | Sorted makes overlaps adjacent |
| "insert interval into sorted list" | Find position, merge neighbors | Check left and right for overlaps |
| "minimum rooms/workers needed" | Sort + min heap of end times | Heap tracks earliest free resource |
| "can attend all meetings" | Sort by start, check consecutive | If any overlap → can't attend all |
| "find free time" | Merge events, find gaps | Gaps between merged events = free |

---

### Template 1 — Merge Intervals

```
Input:    [[1,3],[2,6],[8,10]]
Question: Merge all overlapping intervals
Output:   [[1,6],[8,10]]
```

```python
intervals.sort(key=lambda x: x[0])
merged = [intervals[0]]

for start, end in intervals[1:]:
    if start <= merged[-1][1]:
        merged[-1][1] = max(merged[-1][1], end)
    else:
        merged.append([start, end])
return merged
```

```
- start with [1,3]
- [2,6]: 2<=3 → merge → [1,6]
- [8,10]: 8>6 → add → [[1,6],[8,10]] ✓
```

---

### Template 2 — Minimum Meeting Rooms

```
Input:    [[1,4],[2,5],[7,9]]
Question: Minimum rooms needed?
Output:   2
```

```python
import heapq
intervals.sort(key=lambda x: x[0])
heap = []

for start, end in intervals:
    if heap and heap[0] <= start:
        heapq.heapreplace(heap, end)
    else:
        heapq.heappush(heap, end)
return len(heap)
```

```
- [1,4]: heap=[] → push → heap=[4]
- [2,5]: 2<4 → new room → heap=[4,5]
- [7,9]: 7>4 → reuse → heap=[5,9]
return 2 ✓
```

**Complexity:** O(n log n) time, O(n) space

---
---

# 14. Bit Manipulation

### Mental Trigger
> *"Can I solve this using binary representations — XOR, AND, OR, or shifts?"*

```
Input:    [2, 3, 2, 4, 4]
Question: Find the element that appears only once
Output:   3

XOR:
- 0^2=2
- 2^3=1
- 1^2=3
- 3^4=7
- 7^4=3 ← pairs cancel, unique remains
```

**Why this works:** x XOR x = 0. Identical pairs cancel. Lone value remains.

---

### When You'll See This In Interviews

**Example 1 — Data integrity**
*"Find the one corrupted packet ID where every other ID appears exactly twice."*

```
Input:    [4,1,2,1,2]
Question: Which ID appears only once?
Output:   4

- 0^4=4
- 4^1=5
- 5^2=7
- 7^1=6
- 6^2=4 ✓
```

---

**Example 2 — Permission system**
*"Represent permissions as a single integer where each bit = one permission."*

```
Input:    flags=0, set permission 2
Question: How to set, check, remove a specific permission bit?

set:   flags |= (1<<2)  → flags=4  (100 in binary)
check: (flags>>2) & 1   → 1 (permission is set)
remove:flags &= ~(1<<2) → flags=0
```

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "find the one unique number" | XOR all elements | Pairs cancel, unique remains |
| "check if power of 2" | n & (n-1) == 0 | Powers of 2 have exactly one bit set |
| "count set bits" | n & (n-1) loop | Each op clears one set bit |
| "set/check/remove a flag" | OR/AND/AND-NOT with mask | Bit at position k = 1<<k |

---

### Template 1 — XOR Unique Element

```
Input:    [2, 3, 2, 4, 4]
Question: Find element appearing once
Output:   3
```

```python
result = 0
for num in nums:
    result ^= num
return result
```

```
- 0^2=2, 2^3=1, 1^2=3, 3^4=7, 7^4=3 ✓
```

---

### Template 2 — Count Set Bits

```
Input:    13 (binary: 1101)
Question: How many bits are set?
Output:   3
```

```python
count = 0
while n:
    n &= (n - 1)
    count += 1
return count
```

```
- 1101 & 1100 = 1100 → count=1
- 1100 & 1011 = 1000 → count=2
- 1000 & 0111 = 0000 → count=3 ✓
```

---

### Template 3 — Bit Masking

```python
flags |= (1 << k)          # set bit k
is_set = (flags >> k) & 1  # check bit k
flags &= ~(1 << k)         # clear bit k
is_power_of_2 = n > 0 and (n & (n-1)) == 0
```

**Complexity:** O(1) most operations, O(log n) counting set bits

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

sorted(freq.items(), key=lambda x: x[1], reverse=True)

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
