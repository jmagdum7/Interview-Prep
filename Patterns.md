# Patterns.md

> The goal is pattern recognition, not problem memorization.
> Read this until applying these feels automatic.

---

## How to use this doc

1. Read the pattern — understand the *why*, not just the code
2. Study the real-world examples — train yourself to decode ambiguous problem statements
3. Trace through the examples by hand — visualization only comes from doing, not reading
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
> *"What am I looking up repeatedly? Store that."*

**Example:** checking if any two numbers sum to a target. Brute force scans the array for each number — O(n²). Instead, store what you've seen. One pass — O(n).

**Why:** arrays give you O(1) access by index, but finding relationships between elements requires scanning. A hash map trades O(n) space to turn that scan into O(1) lookup.

---

### When You'll See This In Interviews

**Example 1 — User deduplication**
*"We have millions of user signups. Some users signed up multiple times with the same email. Flag the duplicates."*
→ Hash set. Store emails as you scan. If you see one already in the set — duplicate.

**Example 2 — Transaction matching**
*"Given a list of transactions, find two transactions that together total a specific reimbursement amount."*
→ Complement map. For each transaction, check if `target - amount` was already seen.

**Example 3 — Analytics grouping**
*"Group all search queries that are rearrangements of the same letters — they represent the same user intent."*
→ Frequency map. Sorted or counted character signature is the key. Same signature = same group.

**Example 4 — Running metrics**
*"Our dashboard needs to show the sum of any time range instantly without re-querying the database."*
→ Prefix sums. Precompute once, answer any range in O(1).

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "duplicate", "seen before", "already exists" | Hash set | O(1) existence check |
| "two things that sum/multiply to target" | Complement map | Store what you've seen, check for the missing piece |
| "count", "frequency", "how many times" | Frequency map | Count as you scan |
| "group by shared property", "anagram" | Map of lists | Same key = same group |
| "sum of range", "subarray sum" | Prefix sum | Precompute once, query in O(1) |
| "top K frequent" | Frequency map + bucket sort | Count then retrieve by rank |

---

### Template 1 — Hash Set

```python
seen = set()
for x in arr:
    if x in seen:      # O(1) — already saw this, act on it
        return True
    seen.add(x)        # first time — store for future checks
return False
```

**Trace — `[1, 2, 3, 1]`:**
```
x=1 → not in {} → seen={1}
x=2 → not in {1} → seen={1,2}
x=3 → not in {1,2} → seen={1,2,3}
x=1 → IN {1,2,3} → return True ✓
```

---

### Template 2 — Frequency Map

```python
freq = {}
for x in arr:
    freq[x] = freq.get(x, 0) + 1  # get current count (0 if new), add 1
```

**Trace — `"anagram"` vs `"nagaram"`:**
```
count "anagram" → {'a':3,'n':1,'g':1,'r':1,'m':1}
count "nagaram" → {'n':1,'a':3,'g':1,'r':1,'m':1}
equal → True ✓
```

---

### Template 3 — Complement Map

```python
seen = {}                              # value → index
for i, x in enumerate(arr):
    complement = target - x            # what we NEED
    if complement in seen:             # already passed it?
        return [seen[complement], i]
    seen[x] = i                        # store for future checks
```

**Trace — `[2,7,11,15]`, target=9:**
```
x=2 → need 7 → not seen → store {2:0}
x=7 → need 2 → 2 IS seen → return [0,1] ✓
```

---

### Template 4 — Prefix Sums

```python
prefix = [0] * (len(arr) + 1)         # extra slot so prefix[0]=0 (empty sum)
for i in range(len(arr)):
    prefix[i+1] = prefix[i] + arr[i]  # running total up to index i

range_sum = prefix[j+1] - prefix[i]   # sum from i to j in O(1)
```

**Trace — `[1,2,3,4]`, sum index 1 to 2:**
```
prefix = [0, 1, 3, 6, 10]
prefix[3] - prefix[1] = 6 - 1 = 5  (2+3=5) ✓
```

---

### Template 5 — Bucket Sort (Top K Frequent)

```python
freq = {}
for n in nums:
    freq[n] = freq.get(n, 0) + 1

bucket = [[] for _ in range(len(nums) + 1)]  # index = frequency
for num, count in freq.items():
    bucket[count].append(num)

result = []
for i in range(len(bucket) - 1, 0, -1):      # right to left = most frequent first
    result.extend(bucket[i])
    if len(result) >= k:
        return result[:k]
```

**Why O(n):** frequency used as direct index — no sorting, no comparisons.

**Complexity:** O(n) time, O(n) space

---
---

# 2. Two Pointers

### Mental Trigger
> *"Can I avoid a nested loop by using two positions that move toward each other or at different speeds?"*

**Example:** find two numbers in a sorted array that sum to a target. Brute force tries every pair — O(n²). Instead, start one pointer at each end. If the sum is too big, move the right pointer left. If too small, move the left pointer right. O(n).

**Why:** two pointers works when the data is sorted or when you need to compare elements from opposite ends. Moving pointers inward eliminates candidates without checking every pair.

---

### When You'll See This In Interviews

**Example 1 — Content moderation**
*"Given a string, check if it reads the same forwards and backwards, ignoring spaces and punctuation."*
→ Two pointers from both ends. Compare characters moving inward. If they ever don't match — not a palindrome.

**Example 2 — Chemistry tool**
*"Given a sorted list of compound weights, find all pairs that together equal a target molecular weight."*
→ Sorted array + two pointers. Start at both ends, move inward based on whether sum is too high or too low.

**Example 3 — Memory optimization**
*"Remove all instances of a value from an array in-place without using extra memory."*
→ Two pointers — one scans forward finding elements to keep, one tracks where to place them.

**Example 4 — Geometry**
*"Given an array of heights representing walls, find two walls that together hold the most water."*
→ Two pointers from both ends. Always move the shorter wall inward — moving the taller one can only decrease the area.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "sorted array", "two numbers sum to target" | Two pointers inward | Eliminate candidates by moving inward |
| "palindrome", "reads same forwards/backwards" | Two pointers from ends | Compare from outside in |
| "in-place removal", "no extra memory" | Slow/fast pointer | One reads, one writes |
| "container with most water", "maximize area" | Two pointers inward | Move the limiting side |
| "three numbers sum to target" | Sort + two pointers inside loop | Fix one, use two pointers for the rest |

---

### Template 1 — Two Pointers Inward (Sorted Array)

```python
left, right = 0, len(arr) - 1
while left < right:
    current_sum = arr[left] + arr[right]
    if current_sum == target:
        return [left, right]       # found it
    elif current_sum < target:
        left += 1                  # sum too small — move left up
    else:
        right -= 1                 # sum too big — move right down
```

**Trace — `[1,3,6,8,11]`, target=9:**
```
left=0(1), right=4(11) → sum=12 > 9 → right--
left=0(1), right=3(8)  → sum=9 == 9 → return [0,3] ✓
```

---

### Template 2 — In-place Write

```python
write = 0                          # position to write next kept element
for read in range(len(arr)):
    if arr[read] != val:           # keep this element
        arr[write] = arr[read]
        write += 1
return write                       # new length
```

**Trace — `[1,2,2,3]`, remove 2:**
```
read=0(1): keep → arr[0]=1, write=1
read=1(2): skip
read=2(2): skip
read=3(3): keep → arr[1]=3, write=2
result: [1,3,...], length=2 ✓
```

---

### Template 3 — Three Sum

```python
arr.sort()
result = []
for i in range(len(arr) - 2):
    if i > 0 and arr[i] == arr[i-1]:  # skip duplicates
        continue
    left, right = i + 1, len(arr) - 1
    while left < right:
        total = arr[i] + arr[left] + arr[right]
        if total == 0:
            result.append([arr[i], arr[left], arr[right]])
            while left < right and arr[left] == arr[left+1]: left += 1
            while left < right and arr[right] == arr[right-1]: right -= 1
            left += 1
            right -= 1
        elif total < 0:
            left += 1
        else:
            right -= 1
return result
```

**Complexity:** O(n²) time, O(1) space (excluding output)

---
---

# 3. Sliding Window

### Mental Trigger
> *"Am I looking for a subarray or substring that satisfies a condition — and can I avoid recomputing everything when the window shifts?"*

**Example:** find the longest substring without repeating characters. Brute force checks every substring — O(n²). Instead, maintain a window with a left and right pointer. Expand right, shrink left when a constraint is violated. O(n).

**Why:** sliding window avoids redundant computation. When you move the window, you only add one element and remove one — you don't recompute the entire window from scratch.

---

### When You'll See This In Interviews

**Example 1 — Network monitoring**
*"Given bandwidth readings per second, find the maximum total bandwidth used in any 5-second window."*
→ Fixed-size sliding window. Maintain a running sum. Add new element, remove oldest, track maximum.

**Example 2 — Ad targeting**
*"Find the shortest sequence of user actions that contains all the required action types we want to target."*
→ Variable window. Expand right until all required types are covered, shrink left to minimize length.

**Example 3 — Fraud detection**
*"Find the longest sequence of transactions where no transaction amount repeats."*
→ Variable window with a set. Expand right, shrink left when a duplicate enters the window.

**Example 4 — DNA analysis**
*"Find all positions where a target gene sequence appears as an anagram within a longer DNA string."*
→ Fixed window with frequency map. Slide across, compare character counts at each position.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "fixed size window", "k consecutive elements" | Fixed sliding window | Add right, remove left, track result |
| "longest subarray/substring with condition" | Variable window expand/shrink | Expand until violated, shrink to fix |
| "shortest subarray containing all of X" | Variable window with counter | Track how many conditions are satisfied |
| "no repeating elements in window" | Variable window with set | Set tracks current window contents |
| "anagram in string", "permutation in string" | Fixed window with freq map | Compare frequency maps as you slide |

---

### Template 1 — Fixed Size Window

```python
window_sum = sum(arr[:k])          # sum of first window
max_sum = window_sum

for i in range(k, len(arr)):
    window_sum += arr[i]           # add new right element
    window_sum -= arr[i - k]      # remove old left element
    max_sum = max(max_sum, window_sum)

return max_sum
```

**Trace — `[2,1,5,1,3,2]`, k=3:**
```
initial window: [2,1,5] → sum=8
i=3: add 1, remove 2 → sum=7
i=4: add 3, remove 1 → sum=9
i=5: add 2, remove 5 → sum=6
max = 9 ✓
```

---

### Template 2 — Variable Window

```python
left = 0
seen = set()
max_len = 0

for right in range(len(s)):
    while s[right] in seen:        # constraint violated — shrink
        seen.remove(s[left])
        left += 1
    seen.add(s[right])             # expand window
    max_len = max(max_len, right - left + 1)

return max_len
```

**Trace — `"abcac"`:**
```
right=0(a): seen={a}, len=1
right=1(b): seen={a,b}, len=2
right=2(c): seen={a,b,c}, len=3
right=3(a): 'a' in seen → remove a, left=1 → seen={b,c,a}, len=3
right=4(c): 'c' in seen → remove b,c, left=3 → seen={a,c}, len=2
max = 3 ✓
```

**Complexity:** O(n) time, O(k) space where k = window size

---
---

# 4. Stack

### Mental Trigger
> *"Do I need to remember previous elements in order — and might I need to undo or revisit the most recent one?"*

**Example:** validating that brackets are properly nested. You need to remember the most recent open bracket to match it against the next close bracket. Last in, first out — that's a stack.

**Why:** a stack gives you O(1) access to the most recently seen element. Whenever a problem involves matching, nesting, or undoing in reverse order of how things happened — stack.

---

### When You'll See This In Interviews

**Example 1 — Code editor**
*"Build a feature that validates whether all opening tags in an HTML document have matching closing tags in the right order."*
→ Stack. Push opening tags, pop and match when you see a closing tag. If stack is empty at end — valid.

**Example 2 — Financial data**
*"For each day's stock price, find the number of consecutive previous days where the price was lower."*
→ Monotonic stack. Maintain a stack of decreasing prices. When you see a higher price, pop until the stack is valid again.

**Example 3 — Expression evaluator**
*"Build a calculator that handles nested parentheses and respects operator precedence."*
→ Stack. Push numbers and operators, evaluate when you see a closing parenthesis.

**Example 4 — Navigation history**
*"Implement a browser's back button — the user can undo their last N page visits."*
→ Stack. Push each page visited. Pop to go back.

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

```python
stack = []
matching = {')': '(', '}': '{', ']': '['}

for char in s:
    if char in '({[':
        stack.append(char)          # push opening bracket
    elif char in ')}]':
        if not stack or stack[-1] != matching[char]:
            return False            # no match — invalid
        stack.pop()                 # matched — pop

return len(stack) == 0              # valid only if stack is empty
```

**Trace — `"({[]})"`:**
```
( → push → stack=['(']
{ → push → stack=['(', '{']
[ → push → stack=['(', '{', '[']
] → matches [ → pop → stack=['(', '{']
} → matches { → pop → stack=['(']
) → matches ( → pop → stack=[]
stack empty → True ✓
```

---

### Template 2 — Monotonic Stack (Next Greater Element)

```python
stack = []                          # stores indices
result = [-1] * len(arr)            # default: no greater element

for i in range(len(arr)):
    while stack and arr[i] > arr[stack[-1]]:  # current breaks order
        idx = stack.pop()
        result[idx] = arr[i]        # current is the next greater for idx
    stack.append(i)

return result
```

**Trace — `[2,1,3,4]`:**
```
i=0(2): stack=[0]
i=1(1): 1<2, push → stack=[0,1]
i=2(3): 3>1 → result[1]=3, pop. 3>2 → result[0]=3, pop. push → stack=[2]
i=3(4): 4>3 → result[2]=4, pop. push → stack=[3]
result = [3,3,4,-1] ✓
```

**Complexity:** O(n) time, O(n) space

---
---

# 5. Binary Search

### Mental Trigger
> *"Is the input sorted — or can I define a condition where everything to the left fails and everything to the right passes?"*

**Example:** find a target in a sorted array. Brute force scans every element — O(n). Instead, check the middle. Target smaller? Search left half. Target bigger? Search right half. Eliminate half the options each step — O(log n).

**Why:** binary search works whenever you can eliminate half the search space at each step. It doesn't require a literally sorted array — just a property that's monotonic (once true, always true going in one direction).

---

### When You'll See This In Interviews

**Example 1 — Content delivery**
*"We have server logs sorted by timestamp. Find the first error that occurred after a given time."*
→ Binary search on sorted timestamps. Find leftmost timestamp greater than the given time.

**Example 2 — Resource allocation**
*"Given a list of package weights, find the minimum ship capacity to deliver all packages within D days."*
→ Binary search on the answer. The capacity is between max(weights) and sum(weights). Check if a given capacity works, binary search for the minimum.

**Example 3 — Deployment pipeline**
*"You have 100 versions of software. One version introduced a bug. All versions after it are also broken. Find the first broken version with minimum API calls."*
→ Classic binary search. Check middle version — broken? search left half. Fine? search right half.

**Example 4 — Database indexing**
*"Find the position to insert a new record in a sorted index to maintain sort order."*
→ Binary search for insertion point — leftmost position where element would fit.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "sorted array", "find target" | Classic binary search | Eliminate half each step |
| "first/last occurrence" | Binary search with boundary tracking | Don't stop at first match, keep searching |
| "minimum/maximum that satisfies condition" | Binary search on answer | Check if mid works, search accordingly |
| "find broken version", "first bad commit" | Binary search on condition | Monotonic — once broken, stays broken |
| "rotated sorted array" | Modified binary search | One half is always sorted, use that to decide |

---

### Template 1 — Classic Binary Search

```python
left, right = 0, len(arr) - 1

while left <= right:
    mid = (left + right) // 2      # integer division
    if arr[mid] == target:
        return mid                  # found it
    elif arr[mid] < target:
        left = mid + 1              # target in right half
    else:
        right = mid - 1             # target in left half

return -1                           # not found
```

**Trace — `[1,3,5,7,9]`, target=7:**
```
left=0, right=4, mid=2(5): 5<7 → left=3
left=3, right=4, mid=3(7): 7==7 → return 3 ✓
```

---

### Template 2 — Binary Search on Answer

```python
left, right = min_possible, max_possible

while left < right:
    mid = (left + right) // 2
    if condition(mid):             # mid satisfies condition
        right = mid                # try smaller — find minimum
    else:
        left = mid + 1             # mid doesn't work — go bigger

return left
```

**When to use:** *"find minimum X such that condition holds"* — binary search between the smallest and largest possible answer.

---

### Template 3 — Find Leftmost / Rightmost Occurrence

```python
left, right = 0, len(arr) - 1
result = -1

while left <= right:
    mid = (left + right) // 2
    if arr[mid] == target:
        result = mid               # record this, keep searching
        right = mid - 1            # go left for first occurrence
        # OR: left = mid + 1       # go right for last occurrence
    elif arr[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return result
```

**Complexity:** O(log n) time, O(1) space

---
---

# 6. Linked List

### Mental Trigger
> *"Is the data naturally sequential where each item only knows what comes next — and do I need to manipulate connections rather than values?"*

**Example:** reversing a playlist order without creating a new list. You don't copy — you redirect each song's "next" pointer backwards. That's pointer manipulation on a linked list.

**Why:** linked lists have no indices — you can't jump to position 5. But you can redirect pointers in O(1). Every linked list problem reduces to: which pointers do I move, and in what order?

---

### When You'll See This In Interviews

**Example 1 — Music queue**
*"Users build a queue of songs. Let them play the queue in reverse without extra memory."*
→ Pointer reversal. Redirect each node's pointer backwards in one pass. O(1) space.

**Example 2 — Document version control**
*"Our editor needs unlimited undo. Memory is constrained — we can't store full copies."*
→ Linked list with prev pointer. Each state points to the previous. Undo = follow the pointer back.

**Example 3 — Transaction stream**
*"Detect if our payment processor is stuck in a loop — processing the same transactions repeatedly."*
→ Fast/slow pointers. If there's a cycle, the fast pointer laps the slow one — they meet.

**Example 4 — Log analysis**
*"Find the Kth most recent server error without knowing how many total errors exist."*
→ Two pointers gap k apart. When front hits the end, back is exactly k from it.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "reverse without extra memory" | Pointer reversal | Redirect pointers in place |
| "undo", "go back", "previous state" | Linked list + prev | Each node points to what came before |
| "detect loop or cycle" | Fast/slow pointers | They meet only if a cycle exists |
| "find middle without knowing length" | Fast/slow pointers | Fast reaches end when slow is halfway |
| "Kth from end without full scan" | Two pointers gap k apart | Front hits end, back is k behind |
| "merge two ordered streams" | Dummy node + two pointers | Pick smaller node each step |

---

### Template 1 — Reverse a Linked List

**Why:** save `next_node` first — the moment you redirect `curr.next`, you lose the rest of the list permanently.

```python
prev, curr = None, head
while curr:
    next_node = curr.next    # save rest before redirecting
    curr.next = prev         # reverse the pointer
    prev = curr              # slide prev forward
    curr = next_node         # slide curr forward
return prev                  # prev is the new head
```

**Trace — `1 → 2 → 3 → 4 → None`:**
```
INITIAL:
prev      = None
curr      = 1 → 2 → 3 → 4 → None

STEP 1:
next_node = 2 → 3 → 4 → None
curr.next = None
prev      = 1 → None
curr      = 2 → 3 → 4 → None

STEP 2:
next_node = 3 → 4 → None
curr.next = 1 → None
prev      = 2 → 1 → None
curr      = 3 → 4 → None

STEP 3:
next_node = 4 → None
curr.next = 2 → 1 → None
prev      = 3 → 2 → 1 → None
curr      = 4 → None

STEP 4:
next_node = None
curr.next = 3 → 2 → 1 → None
prev      = 4 → 3 → 2 → 1 → None
curr      = None → loop ends

RETURN prev = 4 → 3 → 2 → 1 → None ✓
```

---

### Template 2 — Fast/Slow Pointers

**Why:** fast moves 2x as fast. When fast reaches the end, slow is exactly halfway. In a cycle, fast laps slow — impossible without a cycle.

```python
slow, fast = head, head
while fast and fast.next:
    slow = slow.next         # 1 step
    fast = fast.next.next    # 2 steps
return slow                  # middle of list
```

**Trace — `1 → 2 → 3 → 4 → 5`:**
```
INITIAL:
slow = 1,  fast = 1

STEP 1: slow=2,  fast=3
STEP 2: slow=3,  fast=5
STEP 3: fast.next=None → stop

slow = 3 ← middle ✓
```

**Cycle detection:**
```python
slow, fast = head, head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:         # met — cycle exists
        return True
return False
```

---

### Template 3 — Kth Node From End

**Why:** gap k between pointers means when right hits `None`, left is exactly k from the end. One pass — no length needed.

```python
left, right = head, head
for _ in range(k):
    right = right.next       # move right k ahead

while right:
    left = left.next
    right = right.next

return left
```

**Trace — `1 → 2 → 3 → 4 → 5`, k=2:**
```
After gap setup:
left  = 1 → 2 → 3 → 4 → 5
right = 3 → 4 → 5 → None

STEP 1: left=2, right=4
STEP 2: left=3, right=5
STEP 3: left=4, right=None → stop

left = 4 ← 2nd from end ✓
```

---

### Template 4 — Dummy Node

**Why:** when the head might change, you need a fixed anchor. Build after dummy, return `dummy.next`.

```python
dummy = ListNode(0)
dummy.next = head
curr = dummy

# manipulate via curr...

return dummy.next
```

**Complexity:** O(n) time, O(1) space

---
---

# 7. Trees

### Mental Trigger
> *"Is the data hierarchical with branching parent-child relationships — and does the same logic apply at every level?"*

**Example:** finding total storage used by a folder including all subfolders. You can't know the folder's total until you know every subfolder's total. Go deep first, combine on the way back up. That's DFS on a tree.

**Why:** trees are naturally recursive. Every node is the root of its own subtree. Whatever you solve for a small tree, you solve the same way for a big one.

---

### When You'll See This In Interviews

**Example 1 — File system**
*"Calculate total storage used by a folder including all nested subfolders and files."*
→ DFS. Go all the way down to files (leaves), return their sizes, sum up on the way back. Root gets the total last.

**Example 2 — Org chart**
*"Find the longest chain of command from CEO to any employee."*
→ DFS for height. Can't know the longest chain from the CEO until you know the longest chain below every manager.

**Example 3 — Notification system**
*"Send a notification to all employees at the same seniority level simultaneously, top down."*
→ BFS. Process all nodes at depth 0, then depth 1, then depth 2 — level by level, naturally.

**Example 4 — GPS routing**
*"Find the fewest road changes to get from location A to location B in a city map."*
→ BFS. Shortest path = fewest steps = BFS, which explores all paths of length 1 before length 2.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "total", "sum", "aggregate across all" | DFS bottom-up | Need leaf values first, combine upward |
| "longest", "deepest", "height", "diameter" | DFS with return values | Go deep, build answer on way back |
| "validate", "compare two trees", "mirror" | DFS recursive | Same check at every node |
| "level by level", "same depth", "layer" | BFS | Queue processes level by level naturally |
| "shortest path", "fewest steps" | BFS | First time you reach destination = shortest |
| "hierarchy", "org chart", "file system" | Tree + DFS or BFS | Branching parent-child = tree |

---

### DFS vs BFS — When and Why

**Use DFS when** the answer requires going all the way down first. You need information from the leaves to answer the question at the root. The call stack takes you deep automatically — leaves return first, parents combine, root gets the final answer.

**Use BFS when** the answer is about levels or proximity to the root. A queue naturally processes all nodes at depth 1 before depth 2. Use when you need level-by-level ordering or shortest path.

---

### Template 1 — DFS Recursive

**Why:** each call handles one node. Ask left child for its answer. Ask right child. Combine. Base case stops at leaves.

```python
def dfs(node):
    if not node:               # past a leaf — return neutral value
        return 0

    left  = dfs(node.left)    # answer from left subtree
    right = dfs(node.right)   # answer from right subtree

    return 1 + max(left, right)  # combine — height example
```

**Trace — height of:**
```
        1
       / \
      2   3
     / \
    4   5
```
```
dfs(1) waiting...
├── dfs(2) waiting...
│   ├── dfs(4): dfs(None)→0, dfs(None)→0 → returns 1
│   └── dfs(5): dfs(None)→0, dfs(None)→0 → returns 1
│   dfs(2): 1 + max(1,1) = 2
└── dfs(3): dfs(None)→0, dfs(None)→0 → returns 1

dfs(1): 1 + max(2,1) = 3 ✓
```

**Key insight:** answer builds bottom-up. You never think about the whole tree — just one node at a time.

---

### Template 2 — BFS Level by Level

**Why:** queue is FIFO. Root in first, out first. Its children added, processed before grandchildren. Naturally level by level.

```python
from collections import deque

queue = deque([root])
while queue:
    node = queue.popleft()     # oldest node first

    if node.left:
        queue.append(node.left)
    if node.right:
        queue.append(node.right)
```

**Trace — same tree:**
```
INITIAL: queue=[1]

pop 1 → add 2,3  → queue=[2,3]
pop 2 → add 4,5  → queue=[3,4,5]
pop 3 → no kids  → queue=[4,5]
pop 4 → no kids  → queue=[5]
pop 5 → no kids  → queue=[]

Order: 1, 2, 3, 4, 5 (level by level) ✓
```

**Why deque not list:** `list.pop(0)` is O(n). `deque.popleft()` is O(1). Always use deque for BFS.

**Complexity:** O(n) time. O(h) space DFS where h=height. O(n) space BFS worst case.

---
---

# 8. Heap / Priority Queue

### Mental Trigger
> *"Do I need to repeatedly find the smallest or largest element from a changing collection?"*

**Example:** find the top K most frequent elements from a stream. You can't sort the whole stream — it keeps growing. Instead, maintain a heap of size K. Every new element challenges the smallest in the heap. O(n log k).

**Why:** a heap gives you O(log n) insert and O(1) access to the min or max. Better than sorting (O(n log n)) when you only need the top K or a running min/max.

---

### When You'll See This In Interviews

**Example 1 — Live leaderboard**
*"We have millions of players. Show the top 10 scores at any moment as new scores come in."*
→ Min heap of size 10. New score beats the minimum in the heap? Replace it. Always maintain the top 10.

**Example 2 — Task scheduling**
*"Given tasks with deadlines and durations, find the order that completes the most tasks before their deadlines."*
→ Min heap on deadlines. Always process the task with the earliest deadline next.

**Example 3 — Data stream median**
*"Find the median of a stream of numbers after each new number arrives."*
→ Two heaps. Max heap for the lower half, min heap for the upper half. Keep them balanced.

**Example 4 — Merging sorted feeds**
*"Merge K sorted news feeds into one sorted feed in real time."*
→ Min heap with one element from each feed. Pop the smallest, push the next from that feed.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "top K", "K largest", "K most frequent" | Min heap size K | Eject smallest when new element wins |
| "K smallest" | Max heap size K | Eject largest when new element wins |
| "running median" | Two heaps (max + min) | Balance halves, median is at boundary |
| "merge K sorted lists" | Min heap with K elements | Always pop global minimum next |
| "next task by priority" | Min/max heap | O(1) access to highest priority |

---

### Template 1 — Top K Largest (Min Heap)

```python
import heapq

heap = []
for num in nums:
    heapq.heappush(heap, num)      # push current number
    if len(heap) > k:
        heapq.heappop(heap)        # remove smallest — keep only top K

return heap                         # contains K largest
```

**Trace — `[3,1,4,1,5,9,2,6]`, k=3:**
```
push 3 → [3]
push 1 → [1,3]
push 4 → [1,3,4] → size=3, no pop
push 1 → [1,1,3,4] → pop 1 → [1,3,4]
push 5 → [1,3,4,5] → pop 1 → [3,4,5]
push 9 → [3,4,5,9] → pop 3 → [4,5,9]
push 2 → [2,4,5,9] → pop 2 → [4,5,9]
push 6 → [4,5,6,9] → pop 4 → [5,6,9]
result: [5,6,9] ← top 3 ✓
```

---

### Template 2 — Running Median (Two Heaps)

```python
import heapq

small = []  # max heap (negate values) — lower half
large = []  # min heap — upper half

def add_num(num):
    heapq.heappush(small, -num)           # push to lower half
    if small and large and (-small[0] > large[0]):
        heapq.heappush(large, -heapq.heappop(small))  # rebalance

    if len(small) > len(large) + 1:
        heapq.heappush(large, -heapq.heappop(small))
    if len(large) > len(small):
        heapq.heappush(small, -heapq.heappop(large))

def find_median():
    if len(small) > len(large):
        return -small[0]
    return (-small[0] + large[0]) / 2
```

---

### Template 3 — Merge K Sorted Lists

```python
import heapq

heap = []
for i, lst in enumerate(lists):
    if lst:
        heapq.heappush(heap, (lst[0], i, 0))  # (value, list_idx, element_idx)

result = []
while heap:
    val, i, j = heapq.heappop(heap)
    result.append(val)
    if j + 1 < len(lists[i]):
        heapq.heappush(heap, (lists[i][j+1], i, j+1))

return result
```

**Complexity:** O(n log k) time where n=total elements, k=number of lists

---
---

# 9. Backtracking

### Mental Trigger
> *"Do I need to explore all possible combinations, permutations, or paths — and can I prune dead ends early?"*

**Example:** generate all possible subsets of a list. You can't do this with a loop — you need to try including and excluding each element at every step. Try a path, if it doesn't work, undo and try another. That's backtracking.

**Why:** backtracking is structured brute force. Instead of trying every possibility blindly, you build solutions incrementally and cut off paths the moment they can't lead to a valid answer.

---

### When You'll See This In Interviews

**Example 1 — Access control**
*"Generate all possible permission combinations a user can have given a set of available permissions."*
→ Subsets. At each permission, try including it and excluding it. Recurse both ways.

**Example 2 — Route planning**
*"Find all possible routes between two cities given a set of available roads."*
→ Path finding with backtracking. Try each road, recurse, undo if it leads to a dead end.

**Example 3 — Puzzle solving**
*"Solve a Sudoku board — fill in missing numbers so each row, column and box has 1-9 exactly once."*
→ Backtracking. Try a number, recurse to fill next cell, undo if you reach a contradiction.

**Example 4 — Scheduling**
*"Given N tasks and N time slots, find all valid ways to assign tasks to slots with no conflicts."*
→ Permutations with constraints. Try each task in each slot, backtrack when a conflict is found.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "all subsets", "power set" | Backtrack include/exclude | Binary choice at each element |
| "all permutations" | Backtrack with used[] | Try each unused element at each position |
| "all combinations that sum to target" | Backtrack with running sum | Prune when sum exceeds target |
| "solve puzzle", "fill grid" | Backtrack with validity check | Try, recurse, undo on contradiction |
| "all paths in a graph/tree" | DFS with path tracking | Explore, add to path, remove on return |

---

### Template 1 — Subsets

```python
def backtrack(start, current):
    result.append(current[:])       # every state is a valid subset

    for i in range(start, len(nums)):
        current.append(nums[i])     # include nums[i]
        backtrack(i + 1, current)   # recurse with remaining
        current.pop()               # undo — try without nums[i]

result = []
backtrack(0, [])
return result
```

**Trace — `[1,2,3]`:**
```
backtrack(0,[])  → add []
  include 1 → backtrack(1,[1]) → add [1]
    include 2 → backtrack(2,[1,2]) → add [1,2]
      include 3 → backtrack(3,[1,2,3]) → add [1,2,3]
    exclude 3 (pop) → [1,2]
    include 3 → backtrack(3,[1,3]) → add [1,3]
  exclude 2 (pop) → [1]
  include 3 → backtrack(3,[1,3])... (already shown)
exclude 1 (pop) → []
... continues for 2 and 3 alone
```

---

### Template 2 — Permutations

```python
def backtrack(current):
    if len(current) == len(nums):
        result.append(current[:])   # complete permutation
        return

    for num in nums:
        if num in current:          # skip used elements
            continue
        current.append(num)
        backtrack(current)
        current.pop()               # undo

result = []
backtrack([])
return result
```

---

### Template 3 — Combination Sum (with pruning)

```python
def backtrack(start, current, remaining):
    if remaining == 0:
        result.append(current[:])   # found valid combination
        return
    if remaining < 0:
        return                      # pruned — sum exceeded target

    for i in range(start, len(candidates)):
        current.append(candidates[i])
        backtrack(i, current, remaining - candidates[i])
        current.pop()

result = []
backtrack(0, [], target)
return result
```

**Complexity:** O(2^n) time worst case — exponential, but pruning makes it much faster in practice

---
---

# 10. Graphs

### Mental Trigger
> *"Is the data a network of connected nodes where I need to traverse connections or find paths?"*

**Example:** find all islands in a grid of land and water. Each cell is a node. Adjacent same-type cells are connected. You need to find connected components — groups of cells that are all reachable from each other. That's graph traversal.

**Why:** graphs model anything with connections — social networks, road maps, dependencies, grids. DFS explores as deep as possible before backtracking. BFS explores neighbors first, guaranteeing shortest path.

---

### When You'll See This In Interviews

**Example 1 — Social network**
*"Find all users who are connected to a given user within 2 degrees of separation."*
→ BFS from that user. Level 1 = direct connections, level 2 = their connections. Stop at depth 2.

**Example 2 — Build system**
*"Given a list of software packages and their dependencies, find a valid build order."*
→ Topological sort. Build dependency graph, process nodes with no remaining dependencies first.

**Example 3 — Mapping**
*"Count the number of distinct islands in a map represented as a grid."*
→ DFS/BFS from each unvisited land cell. Mark all connected land cells as visited. Each DFS = one island.

**Example 4 — Infection spread**
*"Given a grid of people, some infected, find how many days until infection spreads to everyone — or if it can't."*
→ Multi-source BFS. Start BFS from all infected people simultaneously. Each level = one day.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "connected components", "count islands" | DFS/BFS + visited set | Explore each component fully, mark visited |
| "shortest path", "fewest steps" | BFS | Level = distance, first reach = shortest |
| "can reach", "is path possible" | DFS/BFS | Explore from source, check if target found |
| "dependency order", "valid ordering" | Topological sort | Process nodes with no remaining dependencies |
| "spread", "infection", "multi-source" | Multi-source BFS | Start BFS from all sources simultaneously |
| "detect cycle" | DFS with color marking | Back edge = cycle |

---

### Template 1 — DFS (Iterative or Recursive)

```python
def dfs(node, visited):
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(neighbor, visited)

visited = set()
dfs(start, visited)
```

**Grid version (count islands):**
```python
def dfs(r, c):
    if r < 0 or r >= rows or c < 0 or c >= cols:
        return                      # out of bounds
    if grid[r][c] != '1':
        return                      # water or visited
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

---

### Template 2 — BFS (Shortest Path)

```python
from collections import deque

queue = deque([start])
visited = {start}
distance = 0

while queue:
    for _ in range(len(queue)):    # process one level at a time
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

return order if len(order) == len(graph) else []  # empty = cycle exists
```

**Complexity:** O(V + E) time where V=vertices, E=edges

---
---

# 11. Dynamic Programming

### Mental Trigger
> *"Am I solving the same subproblem multiple times — and can I build the answer from smaller answers I've already computed?"*

**Example:** count the ways to climb n stairs taking 1 or 2 steps. To reach stair 5, you came from stair 4 or stair 3. To reach stair 4, you came from stair 3 or stair 2. The same subproblems repeat. Store them — don't recompute.

**Why:** without DP, recursive solutions recompute the same subproblems exponentially. DP stores results (memoization top-down, or tabulation bottom-up) and turns exponential into polynomial.

---

### When You'll See This In Interviews

**Example 1 — Pricing optimization**
*"Given items with weights and values, find the maximum value you can carry in a bag with a weight limit."*
→ 0/1 Knapsack. For each item, decide include or exclude. Build answer from smaller weight limits.

**Example 2 — Text autocomplete**
*"Check if a search query can be broken into valid dictionary words."*
→ Word break DP. For each position, check if any valid word ends here and the prefix before it is also breakable.

**Example 3 — Stock trading**
*"Given daily stock prices, find the maximum profit buying and selling with at most K transactions."*
→ DP on states: day × transactions remaining. Build forward from day 1.

**Example 4 — Similarity scoring**
*"Find the minimum number of edits to transform one document into another."*
→ Edit distance DP. Build a 2D table. Each cell = min edits for substrings up to that point.

---

### Sub-pattern 1 — 1D DP (Fibonacci-style)

**Use when:** each answer depends only on a few previous answers.

```python
dp = [0] * (n + 1)
dp[0] = 1                          # base case
dp[1] = 1                          # base case

for i in range(2, n + 1):
    dp[i] = dp[i-1] + dp[i-2]     # combine previous answers

return dp[n]
```

**Trace — climb stairs, n=4:**
```
dp[0]=1, dp[1]=1
dp[2] = dp[1]+dp[0] = 2
dp[3] = dp[2]+dp[1] = 3
dp[4] = dp[3]+dp[2] = 5 ✓
```

---

### Sub-pattern 2 — 2D DP (Grid / Two Sequences)

**Use when:** problem involves two sequences or a grid, and each cell depends on neighbors.

```python
dp = [[0] * (cols + 1) for _ in range(rows + 1)]

for i in range(1, rows + 1):
    for j in range(1, cols + 1):
        if s1[i-1] == s2[j-1]:
            dp[i][j] = dp[i-1][j-1] + 1   # characters match
        else:
            dp[i][j] = max(dp[i-1][j], dp[i][j-1])  # take best previous
```

**Use for:** longest common subsequence, edit distance, string matching.

---

### Sub-pattern 3 — 0/1 Knapsack

**Use when:** you have items to include or exclude, with a capacity constraint.

```python
dp = [0] * (capacity + 1)

for weight, value in items:
    for w in range(capacity, weight - 1, -1):  # reverse — prevent reuse
        dp[w] = max(dp[w], dp[w - weight] + value)

return dp[capacity]
```

**Trace — items=[(2,3),(3,4),(4,5)], capacity=5:**
```
after item(2,3): dp[2]=3, dp[3]=3, dp[4]=3, dp[5]=3
after item(3,4): dp[3]=4, dp[4]=4, dp[5]=max(3,dp[2]+4)=7
after item(4,5): dp[4]=5, dp[5]=max(7,dp[1]+5)=7
result: dp[5] = 7 ✓
```

---

### Sub-pattern 4 — Unbounded Knapsack

**Use when:** items can be reused unlimited times (coin change, rod cutting).

```python
dp = [float('inf')] * (amount + 1)
dp[0] = 0                          # base: 0 coins for amount 0

for coin in coins:
    for w in range(coin, amount + 1):  # forward — allow reuse
        dp[w] = min(dp[w], dp[w - coin] + 1)

return dp[amount] if dp[amount] != float('inf') else -1
```

**Trace — coins=[1,2,5], amount=6:**
```
after coin 1: dp=[0,1,2,3,4,5,6]
after coin 2: dp=[0,1,1,2,2,3,3]
after coin 5: dp=[0,1,1,2,2,1,2]
result: dp[6] = 2 (5+1) ✓
```

---

### Sub-pattern 5 — Interval DP

**Use when:** the problem involves a range and splitting it at different points.

```python
dp = [[0] * n for _ in range(n)]

for length in range(2, n + 1):          # increasing lengths
    for i in range(n - length + 1):
        j = i + length - 1
        for k in range(i, j):           # try all split points
            dp[i][j] = max(dp[i][j], dp[i][k] + dp[k+1][j] + cost(i,j,k))
```

**Use for:** burst balloons, matrix chain multiplication, palindrome partitioning.

---

### Sub-pattern 6 — DP on Strings (Palindromes)

**Use when:** checking or building palindromic subsequences or substrings.

```python
dp = [[False] * n for _ in range(n)]

for i in range(n):
    dp[i][i] = True                    # single char is palindrome

for length in range(2, n + 1):
    for i in range(n - length + 1):
        j = i + length - 1
        if s[i] == s[j] and (length == 2 or dp[i+1][j-1]):
            dp[i][j] = True
```

**Complexity:** O(n²) to O(n³) depending on sub-pattern. O(n) to O(n²) space.

---
---

# 12. Greedy

### Mental Trigger
> *"Can I make the locally best choice at each step without reconsidering — and does that always lead to the global best?"*

**Example:** given coins of denominations [1,5,10,25], make change for 41 cents with fewest coins. At each step, take the largest coin that fits. Never undo. Works here because larger coins divide evenly into smaller ones.

**Why:** greedy works when local optimal choices don't conflict with global optimality. If you ever need to undo a decision or try alternatives, you need backtracking or DP instead.

---

### When You'll See This In Interviews

**Example 1 — Event scheduling**
*"Given a list of meetings with start and end times, find the maximum number of meetings one person can attend."*
→ Greedy by end time. Always pick the meeting that ends earliest — leaves most room for future meetings.

**Example 2 — Resource allocation**
*"Assign tasks to workers to minimize the time until all tasks are done."*
→ Greedy by task length. Assign longest tasks first to prevent a single long task from bottlenecking everything.

**Example 3 — Jump game**
*"Given an array where each element is the max jump length, determine if you can reach the last index."*
→ Greedy. Track the farthest reachable index. If you can always extend it to cover the next position, you can reach the end.

**Example 4 — Bandwidth optimization**
*"Given file sizes and download speeds, minimize total waiting time."*
→ Greedy by size/speed ratio. Always download the file with best ratio first.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "maximum events/meetings to attend" | Sort by end time, greedy pick | Earliest end = most future room |
| "can you reach the end" | Track max reachable index | Extend greedily at each step |
| "minimum platforms/resources needed" | Sort + track overlaps | Count concurrent needs |
| "assign jobs to minimize time" | Sort by duration | Shortest job first reduces average wait |
| "minimum cost to connect all" | Greedy by edge weight (MST) | Always add cheapest valid connection |

---

### Template 1 — Interval Scheduling (Max Non-overlapping)

```python
intervals.sort(key=lambda x: x[1])    # sort by end time
count = 0
last_end = float('-inf')

for start, end in intervals:
    if start >= last_end:              # no overlap with last picked
        count += 1
        last_end = end                 # update last end time

return count
```

**Trace — `[(1,3),(2,4),(3,5)]`:**
```
sorted by end: [(1,3),(2,4),(3,5)]
(1,3): 1 >= -inf → pick, last_end=3, count=1
(2,4): 2 < 3 → skip (overlaps)
(3,5): 3 >= 3 → pick, last_end=5, count=2
result: 2 ✓
```

---

### Template 2 — Jump Game

```python
max_reach = 0

for i in range(len(nums)):
    if i > max_reach:
        return False               # can't reach position i
    max_reach = max(max_reach, i + nums[i])

return True
```

**Trace — `[2,3,1,1,4]`:**
```
i=0: max_reach=max(0,0+2)=2
i=1: max_reach=max(2,1+3)=4
i=2: max_reach=max(4,2+1)=4
i=3: max_reach=max(4,3+1)=4
i=4: max_reach=max(4,4+4)=8
return True ✓
```

**Complexity:** O(n log n) for sorting-based, O(n) for scan-based greedy

---
---

# 13. Intervals

### Mental Trigger
> *"Does the problem involve ranges with start and end points — and do I need to merge, check overlap, or insert?"*

**Example:** given a list of meeting times, find whether a person can attend all meetings. Two meetings overlap if one starts before the other ends. Sort by start time, check each consecutive pair.

**Why:** sorting by start time turns a complex overlap problem into a simple linear scan. Once sorted, you only need to compare each interval with the one directly before it.

---

### When You'll See This In Interviews

**Example 1 — Calendar**
*"Given a person's calendar, find all free time slots in their day."*
→ Sort and merge overlapping events. Gaps between merged events = free time.

**Example 2 — Booking system**
*"Given existing bookings, insert a new booking and return the updated schedule with no overlapping slots."*
→ Insert interval + merge. Find where it fits, merge with any overlapping existing bookings.

**Example 3 — Resource planning**
*"Given a list of tasks with start and end times, find the minimum number of workers needed so no two tasks assigned to the same worker overlap."*
→ Sort by start time, use a min heap of end times. If earliest-ending worker is free, reuse them. Else add new worker.

**Example 4 — Video streaming**
*"Given segments of a video that have been buffered, find gaps that still need to be loaded."*
→ Merge overlapping buffered segments, find gaps between them.

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

### Template 1 — Merge Overlapping Intervals

```python
intervals.sort(key=lambda x: x[0])    # sort by start time
merged = [intervals[0]]

for start, end in intervals[1:]:
    if start <= merged[-1][1]:         # overlaps with last merged
        merged[-1][1] = max(merged[-1][1], end)  # extend
    else:
        merged.append([start, end])    # no overlap — add new

return merged
```

**Trace — `[[1,3],[2,6],[8,10]]`:**
```
sorted: [[1,3],[2,6],[8,10]]
start with [1,3]
[2,6]: 2<=3 → merge → [1,6]
[8,10]: 8>6 → add → [[1,6],[8,10]] ✓
```

---

### Template 2 — Minimum Meeting Rooms

```python
import heapq

intervals.sort(key=lambda x: x[0])    # sort by start
heap = []                              # tracks end times of active meetings

for start, end in intervals:
    if heap and heap[0] <= start:
        heapq.heapreplace(heap, end)   # reuse room — replace earliest end
    else:
        heapq.heappush(heap, end)      # new room needed

return len(heap)
```

**Complexity:** O(n log n) time, O(n) space

---
---

# 14. Bit Manipulation

### Mental Trigger
> *"Can I solve this using binary representations of numbers — XOR, AND, OR, or bit shifting?"*

**Example:** find the one number that appears only once when all others appear twice. XOR all numbers together. Identical numbers cancel out (x XOR x = 0). The lone number remains.

**Why:** bit operations are O(1) and operate on all 32 bits simultaneously. They're faster and use less space than alternatives for problems involving parity, powers of 2, or unique elements.

---

### When You'll See This In Interviews

**Example 1 — Data integrity**
*"Given a list of packet IDs where every ID appears exactly twice except one corrupted packet, find the corrupted one."*
→ XOR all IDs. Pairs cancel. Lone ID remains.

**Example 2 — Permission system**
*"Represent a set of permissions as a single integer where each bit represents one permission."*
→ Bit masking. Set permission: `flags |= (1 << perm)`. Check permission: `flags & (1 << perm)`. Remove: `flags &= ~(1 << perm)`.

**Example 3 — Performance optimization**
*"Check if a number is a power of 2 with a single operation."*
→ `n & (n-1) == 0`. Powers of 2 have exactly one bit set. Subtracting 1 flips all lower bits. AND = 0.

**Example 4 — Network addressing**
*"Count how many bits are set in an IP address mask."*
→ Count set bits (Hamming weight). `n & (n-1)` clears the lowest set bit. Count how many times until 0.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "find the one unique number" | XOR all elements | Pairs cancel, unique remains |
| "check if power of 2" | `n & (n-1) == 0` | Powers of 2 have exactly one bit set |
| "count set bits" | `n & (n-1)` loop | Each op clears one set bit |
| "set/check/remove a flag" | OR / AND / AND-NOT with mask | Bit at position k = 1 << k |
| "divide/multiply by 2" | Right/left shift | Shifting is faster than division |
| "swap two numbers without temp" | XOR swap | x^y^y = x, cancellation property |

---

### Template 1 — XOR (Find Unique Element)

```python
result = 0
for num in nums:
    result ^= num              # identical pairs cancel out (x^x=0)
return result                  # only unique number remains
```

**Trace — `[2,3,2,4,4]`:**
```
0 ^ 2 = 2
2 ^ 3 = 1
1 ^ 2 = 3
3 ^ 4 = 7
7 ^ 4 = 3 ✓  (only 3 appeared once)
```

---

### Template 2 — Count Set Bits

```python
count = 0
while n:
    n &= (n - 1)               # clears the lowest set bit each time
    count += 1
return count
```

**Trace — n=13 (binary: 1101):**
```
1101 & 1100 = 1100 → count=1
1100 & 1011 = 1000 → count=2
1000 & 0111 = 0000 → count=3
result: 3 set bits ✓
```

---

### Template 3 — Bit Masking

```python
# set bit k
flags |= (1 << k)

# check bit k
is_set = (flags >> k) & 1

# clear bit k
flags &= ~(1 << k)

# check if power of 2
is_power_of_2 = n > 0 and (n & (n - 1)) == 0
```

**Complexity:** O(1) for most bit operations, O(log n) for counting set bits

---
---

# Python Mechanics

Things to know cold. These come up in almost every problem.

---

### Dict and Counter

```python
# always use .items() for key-value pairs
freq = {'a': 3, 'b': 1}
for k, v in freq.items():     # k=key, v=value
    print(k, v)
for k in freq:                # keys only — common mistake

# sorting by value
sorted(freq.items(), key=lambda x: x[1], reverse=True)   # descending
sorted(freq.items(), key=lambda x: x[1], reverse=False)  # ascending

# Counter — dict with counting built in
from collections import Counter
freq = Counter("aabbcc")      # {'a':2,'b':2,'c':2}
freq = Counter([1,2,2,3])     # {2:2,1:1,3:1}

# tuple vs dict
('a', 3)    # tuple — what sorted() gives back
{'a': 3}    # dict — what Counter/freq looks like

# check type when confused
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
q.popleft()          # remove left — O(1)  ← use for BFS, not list.pop(0)
```

---

### Heap (Min Heap by Default)

```python
import heapq
heap = []
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappop(heap)          # returns 1 — always min

# max heap — negate values
heapq.heappush(heap, -5)
-heapq.heappop(heap)         # returns 5

# heapify existing list
arr = [3,1,4,1,5]
heapq.heapify(arr)           # O(n) — converts in place
```

---

### Useful Built-ins

```python
# enumerate — index + value
for i, x in enumerate(arr):
    print(i, x)

# zip — pair two lists
for a, b in zip(list1, list2):
    print(a, b)

# sorting with key
arr.sort(key=lambda x: x[1])         # sort list of tuples by second element
sorted(arr, key=lambda x: -x[1])     # descending

# string operations
s.split()                             # split by whitespace
' '.join(words)                       # join list into string
s[::-1]                               # reverse a string
ord('a')                              # character to ASCII (97)
chr(97)                               # ASCII to character ('a')
```

---
---

# Space & Time Tradeoffs

> At Google and Meta, stating complexity is not enough. You need to reason about *why* the tradeoff is worth it.

---

### The Core Tradeoff

Almost every optimization trades space for time.

- **Hash map:** use O(n) extra space to turn O(n) scan into O(1) lookup
- **Prefix sum:** use O(n) extra space to turn O(n) range query into O(1)
- **Memoization:** use O(n) extra space to avoid recomputing subproblems
- **Bucket sort:** use O(n) extra space to avoid O(n log n) comparison sort

---

### How to Articulate It in an Interview

Don't just say *"this is O(n) time and O(n) space."*

Say: *"I'm using a hash map here — that's O(n) extra space, but it eliminates the inner loop and brings time from O(n²) to O(n). Given that memory isn't constrained in this problem, that's the right tradeoff."*

Three parts:
1. What extra space you're using
2. What time improvement it buys
3. Why that tradeoff is justified given the constraints

---

### When Space IS Constrained

If the interviewer says memory is limited, you need alternatives:

| Normally | Space-constrained alternative |
|---|---|
| Hash set for duplicates | Sort first, check adjacent elements |
| Prefix sum array | Recompute ranges with O(1) space |
| Memoization table | Bottom-up DP with rolling array |
| Storing all results | Stream / yield results one at a time |

---

### Follow-up Optimization Questions

When they ask *"can you do better?"* — think in this order:

1. Can I reduce time? Is there a pattern I'm missing?
2. Can I reduce space? Can I compute on the fly instead of storing?
3. Is there a fundamental lower bound? (e.g. you must read every element once = Ω(n))
4. What if input was a billion elements? → streaming, pagination, distributed

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
