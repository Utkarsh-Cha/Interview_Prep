# K Largest Elements in an Array — Complete Study Notes

> **Prerequisite:** This lecture is a *slight variation* of the previous question (Kth largest element). The instructor strongly recommends watching/reading the previous **two** videos first, because the heap-building logic and the code are reused here almost entirely. Only the final step changes.

---

## 1. Lecture Philosophy (Why the lecture is structured this way)

The instructor begins by explaining his teaching approach, and this is not filler — it directly maps to how you should answer in an interview.

- The questions are deliberately taken in **small, incremental variations** of each other.
- The purpose is that concepts get built up slowly and become rock solid. **There is no rush.**
- Each variation changes only a small part of the previous solution, so you learn to *modify* a known pattern rather than memorise a new one.

### The Flow Used in Every Video

This exact flow is written on screen and is the flow you should follow:

```
Flow:
-> Problem Statement
-> Sorting
-> Heap
-> Explanation
-> Code
```

| Stage | What happens |
|---|---|
| Problem Statement | Understand the input, output, and what exactly is being asked |
| Sorting | The brute-force / naive but correct approach |
| Heap | The optimised approach, derived *from* the sorting approach |
| Explanation | Dry run of the heap approach on the example |
| Code | Actual implementation |

---

## 2. Interview Strategy — Why We Always Start With Sorting (Very Important)

The instructor repeatedly writes "Sorting approach" in every video, and explains why.

### On-screen diagram

```
-> Slight variations
-> Develop

F2F -> Laptop
F2F -> Sorting -> K -> (Could Heap) -> Heap -> Optimization
```

### The reasoning

**Case 1 — Online / laptop-based coding round (machine test, online assessment):**
- There is no discussion, only code submission.
- Here you *must* directly write the good/optimal approach. No point discussing sorting.

**Case 2 — Face-to-face (F2F) interview:**
- **Do NOT immediately rush to the heap solution.**
- Follow this ladder:
  1. First say: *"This can be solved by sorting."* Explain the sorting solution.
  2. Then observe the problem's signals: *"The question gives me **K**, and it asks for **largest**. Also, I do not need the whole array sorted."*
  3. Then say: *"Because of these conditions, a heap can be applied here."*
  4. Then present the heap solution as an **optimisation of the sorting solution**.

### Why this matters (the instructor's core point)

- If you jump straight to the heap, the interviewer concludes you have simply **memorised** the question ("you have crammed it / you had already done this question before").
- If you develop it step by step, you demonstrate that:
  - You *found* a correct solution yourself (sorting),
  - You *identified* the inefficiency in it,
  - You *improved* it into the optimal one.
- That shows genuine understanding rather than rote recall.
- The instructor stresses: **this is not just an interview trick — this is genuinely how you should think.** Start from the beginning, develop the solution, then understand and explain the optimisation.

> **Rule of thumb:** Always start from sorting, especially in a face-to-face interview.

---

## 3. Problem Statement

**Return the K largest elements in an array.**

### Input
- An **array** of integers.
- An **integer K**.

### Output
- An **array of integers** containing the K largest elements.
- (Or, if you are just printing, you can directly output those K numbers.)

### Example (used throughout the lecture)

```
arr[] = [ 7 | 10 | 4 | 3 | 20 | 15 ]
K = 3
```

The three largest elements are **20, 15, 10**.

```
O/P: [ 20 | 15 | 10 ]   or   [ 20 | 10 | 15 ]
```

### Critical clarifications (the instructor emphasises both)

1. **K largest ≠ Kth largest.**
   - "Kth largest" = return **only one** element (the 3rd largest = 10).
   - "K largest" = return **all three** of the largest elements (20, 15, 10).
   - Here we must return **all K of them**.

2. **Order does not matter.**
   - `[20, 15, 10]`, `[20, 10, 15]`, or any permutation is acceptable.
   - You are **not** required to return them in descending order.
   - The only requirement: the K biggest elements must be present.

Reasoning shown on screen: the values 20, 15 and 10 are circled in the original array as the three largest.

---

## 4. Approach 1 — Sorting

### Idea
If the array is sorted in ascending order, the largest elements sit at the **end**. So just pick the last K elements.

### Sorted array with indices (on screen)

```
values :  3   4   7   10   15   20
index  :  0   1   2    3    4    5
size   =  6
K      =  3
```

### Which indices do we pick?

We need the **last K elements**, i.e. we walk backwards from the end:

```
Start at : arr[size - 1]   ->  arr[5]  = 20
End at   : arr[size - k]   ->  arr[3]  = 10
```

So we take `arr[5]`, `arr[4]`, `arr[3]` → **20, 15, 10**. ✅

> **Note on a slip in the lecture:** while speaking, the instructor first says `arr[size - 1 - k]` and then immediately corrects himself using the numbers (size = 6, so `6 - 3 = 3`). The correct written form on screen is:
> - **first index to take → `arr[size - 1]`**
> - **last index to take → `arr[size - k]`**
>
> That is, loop `i` from `size - 1` down to `size - k` (inclusive), which is exactly K elements.

### Sorting approach — pseudo code

```cpp
sort(arr, arr + size);                 // ascending order
for (int i = size - 1; i >= size - k; i--) {
    cout << arr[i] << " ";             // prints 20, 15, 10
}
```

### Complexity
| | |
|---|---|
| Time | **O(N log N)** — dominated by the sort |
| Space | **O(1)** (or O(K) if you store the answer in a new array) |

---

## 5. The Key Observation — Where Sorting Wastes Work

This is the bridge from sorting to heap, and it is the most important insight in the lecture.

On screen, after sorting, the instructor **crosses out indices 0, 1, 2** (values `3, 4, 7`) and labels them:

```
3   4   7  |  10   15   20
^^^^^^^^^
Sort X  /  Extra Work
```

### The reasoning
- We only needed the **last 3** elements in sorted position.
- Sorting the whole array also arranged `3, 4, 7` correctly — but **we get zero benefit from that**.
- Those elements will never be part of the answer. Arranging them among themselves is **pure extra work**.

> **Therefore:** we need a data structure that maintains only the "top K" candidates and does *not* waste effort ordering the rest. That structure is the **heap**.

---

## 6. Heap Identification — How to Recognise a Heap Question

The instructor recalls the identification rule taught earlier and writes it on screen:

```
K + Smallest / Largest  ->  Heap
```

### Rule
If a problem gives you:
- a value **K**, **and**
- the words **smallest** or **largest**,

then a **heap** will apply — almost certainly.

### Applying it here
- Is K given? ✅ (K = 3)
- Is "largest" mentioned? ✅ ("K largest elements")
- Do we need the *whole* array sorted? ❌ No.

→ So the sorting solution can be improved into a **heap** solution.

---

## 7. Choosing the Heap Type — Min Heap or Max Heap?

On-screen note:

```
heap -> min / max
Largest  -> min heap
Smallest -> max heap
```

### The rule (memorise this — it feels backwards at first)

| Question asks for | Heap to use |
|---|---|
| K **largest** | **Min heap** |
| K **smallest** | **Max heap** |

Since our question asks for the K **largest**, we use a **min heap**.

### What a min heap means (instructor's definition)

- In a **min heap**, the **smallest element among all elements currently in the heap sits at the top**.
- The rest of the elements can be in any internal order — **only the top is guaranteed** to be the minimum.
- In a **max heap**, the **largest element sits at the top**, and again the remaining order is arbitrary.

### Why min heap works for "K largest" (the intuition)

We keep the heap size capped at K. Whenever the heap overflows to K+1 elements, we must throw one away — and the one we should throw away is the **smallest** of the current candidates, because it is the one least likely to belong to the final K largest.

A **min heap keeps that smallest element right at the top**, so removing it is a single `pop()` operation. That is exactly why "largest" pairs with "min heap".

---

## 8. The Algorithm

1. Create a **min heap**.
2. **Traverse the array** element by element.
3. Push the current element into the min heap.
4. **Check the heap size.** Never let it exceed K:
   - if `heap.size() > k` → **pop** (this removes the current smallest).
5. After the traversal ends, the heap contains exactly **K elements — the K largest of the array**.
6. **Print / return all of them** by emptying the heap.

> The difference from the *previous* question (Kth largest): there, at the end you only took `heap.top()` (a single element). Here, you must output the **entire remaining heap**.

---

## 9. Full Dry Run (Step-by-step, exactly as drawn on screen)

```
arr[] = 7, 10, 4, 3, 20, 15
K = 3
```

| Step | Element pushed | Heap contents (top = smallest) | size | size > K? | Action |
|---|---|---|---|---|---|
| 1 | 7 | `7` | 1 | No | continue |
| 2 | 10 | `7, 10` (7 on top, 10 goes below) | 2 | No | continue |
| 3 | 4 | `4, 7, 10` (4 rises to top) | 3 | No | continue |
| 4 | 3 | `3, 4, 7, 10` (3 rises to top) | 4 | **Yes** | **pop 3** → heap = `4, 7, 10` |
| 5 | 20 | `4, 7, 10, 20` (20 sinks to the bottom) | 4 | **Yes** | **pop 4** → heap = `7, 10, 20` |
| 6 | 15 | `7, 10, 15, 20` (7 stays on top) | 4 | **Yes** | **pop 7** → heap = `10, 15, 20` |

### Final heap state

```
[ 10, 15, 20 ]   ← these are the 3 largest elements ✅
```

### Walk-through in words (matching the lecture)

- **7** is pushed. Size is 1, which is within 3 — nothing wrong, move on.
- **10** is pushed. Since this is a *min* heap, **7 stays on top and 10 goes below**. Size is 2 — fine.
- **4** is pushed. Now the arrangement becomes `4` on top with `7` and `10` beneath. Size is 3 — still not greater than K, so nothing wrong.
- **3** is pushed. Heap now holds `3, 4, 7, 10` with 3 on top. Size = 4 which is `> K`, so we **immediately pop**. The top (3) is removed. Heap = `4, 7, 10`, size back to K.
- **20** is pushed. Because it is a min heap, 20 goes to the bottom (it is the largest). Size = 4 `> K`, so pop the top → **4 is removed**. Heap = `7, 10, 20`.
- **15** is pushed. It settles in, with **7 rising/remaining at the top**. Size = 4 `> K`, so pop → **7 is removed**. Heap = `10, 15, 20`.

---

## 10. The Beautiful Cross-Check (Sorting ↔ Heap)

The instructor ties the two approaches together — this is a great line to say in an interview.

Sorted array (written again on screen as reference):

```
3   4   7  |  10   15   20
```

- In the sorting approach, `3, 4, 7` were identified as **extra work** — elements we sorted but never used.
- In the heap approach, the elements that got **popped** were exactly **3, 4, and 7**.

> **Conclusion:** The heap never does the wasted work. It discards precisely the elements that sorting wasted time on, and keeps only the K candidates that matter. This is *why* the heap is the optimisation of sorting.

---

## 11. Outputting the Answer

### What changes compared to the previous question

- **Previous question (Kth largest):** after the traversal, you only needed `min_heap.top()` — a single value.
- **This question (K largest):** you need **all K elements**, and all of them are sitting inside the heap at the end. So you **empty the entire heap**, printing each top before popping.

### Code written on screen

```cpp
while (min_heap.size() > 0)
{
    cout << min_heap.top() << " ";
    min_heap.pop();
}
```

### Execution of this loop on our final heap `[10, 15, 20]`

| Iteration | `top()` printed | after `pop()` | size |
|---|---|---|---|
| 1 | 10 | `15, 20` | 2 |
| 2 | 15 | `20` | 1 |
| 3 | 20 | `` (empty) | 0 |

Loop condition `size() > 0` fails → loop ends.

**Printed output:** `10 15 20`

This is a valid answer because **order does not matter** (as established in Section 3). Note that popping a min heap naturally gives **ascending** order.

---

## 12. Complete Code

The instructor notes that the first part (traverse → push → size check → pop) is **exactly the same code as the previous video**; only the final output block is added. Here is the whole thing put together:

```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
    int arr[] = {7, 10, 4, 3, 20, 15};
    int size = 6;
    int k = 3;

    // Min heap: greater<int> makes priority_queue behave as a min heap,
    // i.e. the SMALLEST element stays at the top.
    priority_queue<int, vector<int>, greater<int>> min_heap;

    // --- Same as the previous video ---
    for (int i = 0; i < size; i++)
    {
        min_heap.push(arr[i]);          // put current element into the heap

        if (min_heap.size() > k)        // heap must never exceed size K
            min_heap.pop();             // removes the current smallest
    }

    // --- The only NEW part for "K largest" ---
    while (min_heap.size() > 0)
    {
        cout << min_heap.top() << " ";  // print smallest of the remaining K
        min_heap.pop();                 // then remove it
    }

    return 0;
}
```

### Line-by-line logic

| Line | Purpose |
|---|---|
| `priority_queue<int, vector<int>, greater<int>>` | Declares a **min heap**. By default C++ `priority_queue` is a *max* heap, so `greater<int>` is required to flip it. |
| `min_heap.push(arr[i])` | Adds the current array element as a candidate for the top-K. |
| `if (min_heap.size() > k) min_heap.pop()` | The heart of the algorithm. As soon as we have K+1 candidates, the weakest (smallest) one — which is conveniently at the top of a min heap — is discarded. |
| `while (min_heap.size() > 0) { top; pop; }` | Empties the heap, outputting all K surviving elements. |

### If you need to *return* an array instead of printing

```cpp
vector<int> ans;
while (min_heap.size() > 0) {
    ans.push_back(min_heap.top());
    min_heap.pop();
}
return ans;   // contains the K largest elements (ascending order)
```

---

## 13. Complexity Analysis

### Heap approach
| | |
|---|---|
| Time | **O(N log K)** — N elements, each push/pop costs `log K` because the heap never grows beyond K |
| Space | **O(K)** — only K elements are ever stored in the heap |

### Comparison

| Approach | Time | Space | Wasted work? |
|---|---|---|---|
| Sorting | O(N log N) | O(1) | Yes — fully orders elements that can never be in the answer |
| Min heap | **O(N log K)** | O(K) | No — discards non-candidates immediately |

Since K ≤ N, `log K ≤ log N`, so the heap approach is at least as good and much better when K is small relative to N.

---

## 14. Quick Revision Sheet

- **Question:** return all K largest elements (not the Kth — *all* K).
- **Order of output:** doesn't matter.
- **Heap identification signal:** `K + smallest/largest → Heap`.
- **Heap type rule:** largest → **min heap**; smallest → **max heap**.
- **Min heap property:** smallest element at top; rest of the order is unspecified.
- **Core loop:** push element → if `size > K`, pop.
- **Why it works:** popping a min heap's top removes the weakest candidate each time; after one pass, only the K largest survive.
- **Sorting link:** the elements popped by the heap (`3, 4, 7`) are exactly the "extra work" elements that sorting wasted time on.
- **Final step (the only change from the previous question):** instead of returning just `top()`, drain the whole heap with a `while (size > 0)` loop.
- **Interview delivery:** Problem statement → sorting solution → point out the extra work → identify K + largest → heap → choose min heap → dry run → code. **Never jump straight to the heap in a face-to-face interview.**
