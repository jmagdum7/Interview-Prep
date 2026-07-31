# Interview_Walkthrough.md

> How to behave in the room. Read this before every mock session.

---

## Table of Contents

- [The Framework — what to do step by step](#the-framework)
- [What to Master Before Interview Day](#what-to-master-before-interview-day)

---

## The Framework

### Step 1 — Clarify before touching code (2–3 min)

Ask one scoping question immediately. Don't start coding cold.

- "Can the input contain duplicates?"
- "Should I optimize for time or space?"
- "What's the expected input size — does O(n²) matter here?"
- "Can values be negative?"

State your assumptions aloud: *"I'll assume the array is non-empty and values are integers."*

### Step 2 — State the brute force first

Always say the naive solution out loud before optimizing. This shows you understand the problem and gives a baseline.

*"The brute force would be two nested loops — O(n²). We can do better."*

### Step 3 — Identify the pattern

Ask yourself:
1. What am I repeatedly looking up? → hash map/set
2. Do I need two elements that satisfy something? → two pointers or complement trick
3. Is there a window or subarray involved? → sliding window
4. Does order/undo matter? → stack

### Step 4 — Talk through your approach before coding

*"So I'll make one pass, store each element in a hash map with its index, and check for the complement at each step. That gets us to O(n) time."*

Wait for the interviewer to nod or respond before writing code.

### Step 5 — Code cleanly

- Use meaningful variable names — not `i, j, k` everywhere
- Write top-down: function signature → high-level steps as comments → fill in
- Don't erase — cross out or leave notes if you change direction

### Step 6 — Test with an example

Walk through a small input manually after writing code.

*"Let me trace through [2, 7, 11] with target 9..."*

Always test an edge case: empty input, single element, all duplicates.

### Step 7 — State complexity unprompted

Before they ask:
*"This is O(n) time and O(n) space because we're storing at most n elements in the map."*

---

## What to Master Before Interview Day

These are the seven areas that determine whether you pass or fail. Technical knowledge alone is not enough. All seven must be addressed.

---

### 1 — Pattern Recognition

**What ready looks like:** Given any Leetcode problem, you can identify the correct pattern within 2-3 minutes without hints. You are not memorizing solutions — you are recognizing the shape of the problem.

---

### 2 — Real Problem → Pattern → Solution Translation

**What ready looks like:** When given a real-world problem statement ("we have millions of users signing up..."), you can strip away the business language, identify the underlying CS problem, map it to a pattern, and begin solving. This is the most underrated skill in interviews. Most candidates practice clean Leetcode problems. Interviewers give messy real-world descriptions.

---

### 3 — Interview Simulation Under Pressure

**What ready looks like:** You can solve a Medium Leetcode problem in under 35 minutes while talking out loud, with no hints, and with someone watching. Silence under pressure is the most common failure mode. This only gets fixed through repetition — not more studying.

---

### 4 — System Design

**What ready looks like:** Given any system design prompt, you can drive the conversation — clarify scope, identify components, draw the architecture, state tradeoffs, and handle follow-up constraints. You raise tradeoffs before being asked. You handle scale, failure points, and bottlenecks unprompted.

---

### 5 — Behavioral Stories

**What ready looks like:** You have 8-10 strong STAR stories from your own experience, each with concrete metrics, each adaptable to multiple principles. You can deliver any of them in under 2 minutes, out loud, without rambling. One weak behavioral round cancels strong technical performance.

---

### 6 — Mock Interviews

**What ready looks like:** You have completed at least 15 full 45-minute mock sessions before applying. Not practice problems — full simulations with another person, timed, no pausing, no hints. The first 3-4 mocks will be uncomfortable. That discomfort is the point.

---

### 7 — Complexity Analysis Communication

**What ready looks like:** You state time and space complexity unprompted, before the interviewer asks, with a clear explanation of why. *"This is O(n) time because we make one pass, and O(n) space because we store at most n elements in the map."* This is an explicit green flag interviewers look for. Forgetting space complexity is one of the most common slips.
