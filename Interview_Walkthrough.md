# Interview_Walkthrough.md

> How to behave in the room. Read this before every mock session.

---

## Table of Contents

- [The Mindset](#the-mindset)
- [The Framework — what to do step by step](#the-framework)
- [Handling Ambiguity](#handling-ambiguity)
- [Communicating Your Thinking](#communicating-your-thinking)
- [When You're Stuck](#when-youre-stuck)
- [Complexity Analysis — how to talk about it](#complexity-analysis)
- [Green Flags / Red Flags](#green-flags--red-flags)

---

## The Mindset

An interview is not a test. It is a **collaborative problem-solving session** where the interviewer wants to see how you think, not just what you know.

Senior-level interviews deliberately introduce ambiguity. The right move is never to wait — it's to drive.

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

## Handling Ambiguity

Ambiguity in senior interviews is **deliberate**. The interviewer is watching whether you:

- Ask a clarifying question upfront ✓
- State your assumptions aloud ✓
- Check in at transitions ("does this approach make sense before I continue?") ✓
- Wait silently and hope they fill the gap ✗

One good question beats five minutes of silent coding.

---

## Communicating Your Thinking

Narrate as you go. Examples:

- *"I'm using a set here because I only need existence, not the count."*
- *"I'm iterating from the right because..."*
- *"This feels like it could be O(n²) — let me think if there's a better way."*

If you realize you're going down the wrong path, say so:
*"Actually, this doesn't handle the case where values repeat — let me reconsider."*

Pivoting out loud is a green flag. Silently scrapping and restarting is a red flag.

---

## When You're Stuck

Don't go silent. Say:

1. *"Let me think through the brute force first."*
2. *"What if I sorted the input first — would that help?"*
3. *"I'm thinking about this pattern but I'm not sure it applies here — can I talk through it?"*

Interviewers will often give hints if you're visibly reasoning. They give nothing if you're silent.

---

## Complexity Analysis

### How to think about it

| Situation | Complexity |
|---|---|
| Single loop over n elements | O(n) |
| Two nested loops | O(n²) |
| Loop + hash map lookup inside | O(n) — lookup is O(1) |
| Sorting | O(n log n) |
| Binary search | O(log n) |

### How to say it

*"Time complexity is O(n) — we make one pass through the array, and each hash map operation is O(1). Space is O(n) in the worst case because we store every element."*

Always mention both time **and** space. Forgetting space is a common slip.

---

## Green Flags / Red Flags

| Green flag | Red flag |
|---|---|
| Asks a clarifying question first | Starts coding immediately |
| States brute force before optimizing | Jumps to optimal without explaining why |
| Narrates thinking throughout | Goes silent for minutes |
| Tests with an example | Submits without tracing through |
| Pivots out loud when wrong | Silently erases and restarts |
| States complexity unprompted | Waits to be asked |
| Checks in at transitions | Never engages the interviewer |
