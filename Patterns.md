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
> *"What am I repeatedly looking up? Store that."*

```
Input:    [2, 7, 11, 15], target=9
Question: Do any two numbers sum to the target?
Output:   [0, 1] → (2 + 7 = 9)

Brute force:
- For 2 → scan [7,11,15] for 7 → found (2 loops)
- For 7 → scan [11,15] for 2 → already passed
- O(n²)

Hash map:
- For 2 → need 7 → not in map → store {2:0}
- For 7 → need 2 → 2 IS in map → return [0,1]
- O(n)
```

**Why this works:** instead of scanning for the pair, you store what you've seen and look it up instantly.

---

### When You'll See This In Interviews

**Example 1 — User deduplication**
*"We have millions of user signups. Some users signed up multiple times with the same email. Flag the duplicates."*

```
Input:    [a@x.com, b@x.com, a@x.com]
Question: Which emails appear more than once?
Output:   a@x.com

- add a@x.com → seen={a@x.com}
- add b@x.com → seen={a@x.com, b@x.com}
- add a@x.com → already in seen! → duplicate found
```

→ Hash set. Store as you scan. If already there — duplicate.

---

**Example 2 — Analytics grouping**
*"Group all search queries that are rearrangements of the same letters."*

```
Input:    ["eat", "tea", "tan", "ate", "nat", "bat"]
Question: Which words are anagrams of each other?
Output:   [["eat","tea","ate"], ["tan","nat"], ["bat"]]

- "eat" → sorted → "aet" → groups={"aet": ["eat"]}
- "tea" → sorted → "aet" → groups={"aet": ["eat","tea"]}
- "tan" → sorted → "ant" → groups={"aet": [...], "ant": ["tan"]}
- "ate" → sorted → "aet" → groups={"aet": ["eat","tea","ate"]}
- "nat" → sorted → "ant" → groups={"ant": ["tan","nat"]}
- "bat" → sorted → "abt" → groups={"abt": ["bat"]}
```

→ Frequency map. Same sorted signature = same group.

---

**Example 3 — Running metrics**
*"Our dashboard needs to show the sum of any time range instantly."*

```
Input:    [1, 2, 3, 4], query: sum from index 1 to 2
Question: What is the sum between any two indices in O(1)?
Output:   5 (2+3)

prefix = [0, 1, 3, 6, 10]
sum(1,2) = prefix[3] - prefix[1] = 6 - 1 = 5
```

→ Prefix sum. Precompute once, answer any range in O(1).

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

```
Input:    [1, 2, 3, 1]
Question: Does any value appear more than once?
Output:   True
```

```python
seen = set()
for x in arr:
    if x in seen:      # O(1) — already saw this, act on it
        return True
    seen.add(x)        # first time — store for future checks
return False
```

```
- add 1 → seen={1}
- add 2 → seen={1,2}
- add 3 → seen={1,2,3}
- add 1 → already in seen! → return True ✓
```

---

### Template 2 — Frequency Map

```
Input:    "anagram", "nagaram"
Question: Are these two strings anagrams?
Output:   True
```

```python
freq = {}
for x in arr:
    freq[x] = freq.get(x, 0) + 1
```

```
count "anagram" → {a:3, n:1, g:1, r:1, m:1}
count "nagaram" → {n:1, a:3, g:1, r:1, m:1}
equal → True ✓
```

---

### Template 3 — Complement Map

```
Input:    [2, 7, 11, 15], target=9
Question: Which two indices sum to the target?
Output:   [0, 1]
```

```python
seen = {}
for i, x in enumerate(arr):
    complement = target - x
    if complement in seen:
        return [seen[complement], i]
    seen[x] = i
```

```
- x=2 → need 7 → not in map → store {2:0}
- x=7 → need 2 → 2 IN map → return [0,1] ✓
```

---

### Template 4 — Prefix Sums

```
Input:    [1, 2, 3, 4], query: sum index 1 to 2
Question: What is the range sum in O(1)?
Output:   5
```

```python
prefix = [0] * (len(arr) + 1)
for i in range(len(arr)):
    prefix[i+1] = prefix[i] + arr[i]

range_sum = prefix[j+1] - prefix[i]
```

```
prefix = [0, 1, 3, 6, 10]
prefix[3] - prefix[1] = 6 - 1 = 5 ✓
```

---

### Template 5 — Bucket Sort (Top K Frequent)

```
Input:    [1,1,1,2,2,3], k=2
Question: What are the top 2 most frequent elements?
Output:   [1, 2]
```

```python
freq = {}
for n in nums:
    freq[n] = freq.get(n, 0) + 1

bucket = [[] for _ in range(len(nums) + 1)]
for num, count in freq.items():
    bucket[count].append(num)

result = []
for i in range(len(bucket) - 1, 0, -1):
    result.extend(bucket[i])
    if len(result) >= k:
        return result[:k]
```

```
freq = {1:3, 2:2, 3:1}
bucket[3]=[1], bucket[2]=[2], bucket[1]=[3]
scan right to left → result=[1,2] → len=2=k → return [1,2] ✓
```

**Complexity:** O(n) time, O(n) space

---
---

# 2. Two Pointers

### Mental Trigger
> *"Can I avoid a nested loop by using two positions that move toward each other or at different speeds?"*

```
Input:    [1, 3, 6, 8, 11], target=9
Question: Do any two numbers sum to the target?
Output:   [0, 3] → (1 + 8 = 9)

Brute force:
- Try every pair → O(n²)

Two pointers:
- left=1, right=11 → sum=12 > 9 → move right left
- left=1, right=8  → sum=9 == 9 → found
- O(n)
```

**Why this works:** sorted array means moving pointers inward eliminates candidates without checking every pair.

---

### When You'll See This In Interviews

**Example 1 — Palindrome check**
*"Check if a string reads the same forwards and backwards, ignoring spaces."*

```
Input:    "racecar"
Question: Is this a palindrome?
Output:   True

- left=r, right=r → match → move inward
- left=a, right=a → match → move inward
- left=c, right=c → match → move inward
- left=e → middle → done → True
```

→ Two pointers from both ends. Compare moving inward.

---

**Example 2 — Container with most water**
*"Given heights of walls, find two walls that hold the most water."*

```
Input:    [1, 8, 6, 2, 5, 4, 8, 3, 7]
Question: What is the maximum water between any two walls?
Output:   49

- left=1, right=7 → area=min(1,7)*8=8 → move left (shorter wall)
- left=8, right=7 → area=min(8,7)*7=49 → move right
- ...continue until pointers meet
```

→ Always move the shorter wall inward — moving the taller one can only decrease area.

---

**Example 3 — Remove duplicates in-place**
*"Remove all instances of a value from an array without extra memory."*

```
Input:    [1, 2, 2, 3], remove 2
Question: Remove target value in-place, return new length
Output:   [1, 3], length=2

- read=0(1): keep → write=0→1, write moves to 1
- read=1(2): skip
- read=2(2): skip
- read=3(3): keep → write=1→3, write moves to 2
result: [1,3,...], length=2
```

→ Slow/fast pointer. One reads, one writes.

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

### Template 1 — Two Pointers Inward

```
Input:    [1, 3, 6, 8, 11], target=9
Question: Which two indices sum to the target?
Output:   [0, 3]
```

```python
left, right = 0, len(arr) - 1
while left < right:
    current_sum = arr[left] + arr[right]
    if current_sum == target:
        return [left, right]
    elif current_sum < target:
        left += 1
    else:
        right -= 1
```

```
- left=1, right=11 → sum=12 > 9 → right--
- left=1, right=8  → sum=9 == 9 → return [0,3] ✓
```

---

### Template 2 — In-place Write

```
Input:    [1, 2, 2, 3], val=2
Question: Remove all 2s in-place, return new length
Output:   2, array=[1,3]
```

```python
write = 0
for read in range(len(arr)):
    if arr[read] != val:
        arr[write] = arr[read]
        write += 1
return write
```

```
- read=0(1): keep → arr[0]=1, write=1
- read=1(2): skip
- read=2(2): skip
- read=3(3): keep → arr[1]=3, write=2
result: [1,3,...], length=2 ✓
```

---

### Template 3 — Three Sum

```
Input:    [-1, 0, 1, 2, -1, -4]
Question: Find all unique triplets that sum to zero
Output:   [[-1,-1,2], [-1,0,1]]
```

```python
arr.sort()
result = []
for i in range(len(arr) - 2):
    if i > 0 and arr[i] == arr[i-1]:
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

```
sorted: [-4,-1,-1,0,1,2]
fix -4 → left=-1, right=2 → sum=-3 < 0 → left++
fix -1 → left=-1, right=2 → sum=0 → found [-1,-1,2]
fix -1 → left=0, right=1 → sum=0 → found [-1,0,1]
```

**Complexity:** O(n²) time, O(1) space

---
---

# 3. Sliding Window

### Mental Trigger
> *"Am I looking for a subarray or substring that satisfies a condition — and can I avoid recomputing everything when the window shifts?"*

```
Input:    [2, 1, 5, 1, 3, 2], k=3
Question: What is the largest sum of any 3 consecutive elements?
Output:   9

Brute force:
- [2,1,5] = 8
- [1,5,1] = 7
- [5,1,3] = 9
- Recalculates every window from scratch

Sliding window:
- window=[2,1,5] → sum=8
- drop 2, add 1 → window=[1,5,1] → sum=7
- drop 1, add 3 → window=[5,1,3] → sum=9
- Adjusts by one element each step
```

**Why this works:** you're not recomputing. You're adjusting.

---

### When You'll See This In Interviews

**Example 1 — Network monitoring**
*"Given bandwidth readings per second, find the maximum total bandwidth used in any 5-second window."*

```
Input:    [3, 1, 4, 1, 5, 9, 2, 6], k=5
Question: What is the highest total bandwidth in any 5-second window?
Output:   23

- window=[3,1,4,1,5] → sum=14
- drop 3, add 9 → window=[1,4,1,5,9] → sum=20
- drop 1, add 2 → window=[4,1,5,9,2] → sum=21
- drop 4, add 6 → window=[1,5,9,2,6] → sum=23
```

→ Fixed window. Add right, remove left, track maximum.

---

**Example 2 — Fraud detection**
*"Find the longest sequence of transactions where no amount repeats."*

```
Input:    [100, 200, 300, 100, 400]
Question: What is the longest sequence with no repeating amount?
Output:   4 → [200, 300, 100, 400]

- add 100 → window=[100]
- add 200 → window=[100,200]
- add 300 → window=[100,200,300]
- add 100 → duplicate! shrink left until removed → window=[200,300,100]
- add 400 → window=[200,300,100,400] → length=4
```

→ Variable window. Expand right, shrink left when constraint breaks.

---

**Example 3 — Ad targeting**
*"Find the shortest sequence of user actions that contains all required action types."*

```
Input:    actions=[A,B,C,A,B], required={A,B,C}
Question: What is the shortest sequence containing all of A, B, C?
Output:   3 → [A,B,C]

- add A → window=[A] → missing {B,C}
- add B → window=[A,B] → missing {C}
- add C → window=[A,B,C] → all covered → length=3
- shrink left → window=[B,C] → missing {A} → stop
```

→ Variable window with counter. Expand to satisfy, shrink to minimize.

---

### Translation Table

| You hear this... | Pattern | Why |
|---|---|---|
| "fixed size window", "k consecutive elements" | Fixed sliding window | Add right, remove left, track result |
| "longest subarray/substring with condition" | Variable window expand/shrink | Expand until violated, shrink to fix |
| "shortest subarray containing all of X" | Variable window with counter | Track how many conditions satisfied |
| "no repeating elements in window" | Variable window with set | Set tracks current window contents |
| "anagram in string", "permutation in string" | Fixed window with freq map | Compare frequency maps as you slide |

---

### Template 1 — Fixed Size Window

```
Input:    [2, 1, 5, 1, 3, 2], k=3
Question: What is the largest sum of any 3 consecutive elements?
Output:   9
```

```python
window_sum = sum(arr[:k])
max_sum = window_sum

for i in range(k, len(arr)):
    window_sum += arr[i]
    window_sum -= arr[i - k]
    max_sum = max(max_sum, window_sum)

return max_sum
```

```
- window=[2,1,5] → sum=8
- drop 2, add 1 → window=[1,5,1] → sum=7
- drop 1, add 3 → window=[5,1,3] → sum=9
- drop 5, add 2 → window=[1,3,2] → sum=6
max=9 ✓
```

**Complexity:** O(n) time, O(1) space

---

### Template 2 — Variable Window

```
Input:    "abcac"
Question: What is the longest substring with no repeating characters?
Output:   3 → "abc"
```

```python
left = 0
seen = set()
max_len = 0

for right in range(len(s)):
    while s[right] in seen:
        seen.remove(s[left])
        left += 1
    seen.add(s[right])
    max_len = max(max_len, right - left + 1)

return max_len
```

```
- add a → window=[a]
- add b → window=[a,b]
- add c → window=[a,b,c] → len=3
- add a → duplicate! shrink left → window=[b,c,a] → len=3
- add c → duplicate! shrink left → window=[a,c] → len=2
max=3 ✓
```

**Complexity:** O(n) time, O(k) space where k = window size

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
Input:    [3,1,4,1,5,9,2,6], k=3
Question: What are the 3 largest elements?
Output:   [5,6,9]
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
- pop 1 → result=[1], push (4,0,1) → heap=[(2,1,0),(3,2,0),(4,0,1)]
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

**Why this works:** structured brute force. Build → explore → undo. Cut off paths that can't work.

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
- try 2 → remaining=5 → try 3 → remaining=2 → try 2 → remaining=0 → found [2,3,2] (skip dup)
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
Output:   3

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
