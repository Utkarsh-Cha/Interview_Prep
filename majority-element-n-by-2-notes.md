# Majority Element (> N/2 times) — Complete Study Notes

**Course context:** Striver's A2Z DSA Course → Step 3: Solve Problems on Arrays → Step 3.2: Medium.
The previous problem covered was *Sort an array of 0's, 1's and 2's*; this lecture covers *Majority Element (> n/2 times)*. Later problems in the same section include Kadane's Algorithm, Stock Buy and Sell, Next Permutation, Leaders in an Array, and Longest Consecutive Sequence.

---

## 1. Problem Statement

You are given an array of integers. Your task is to **find the element that appears more than N/2 times**, where `N` is the length of the array.

### The critical wording: "more than N/2", NOT "equal to N/2"

This single word decides whether your solution is right or wrong, so read it carefully:

- The element must appear **strictly greater than** `N/2` times.
- `N/2` is computed using **integer (floor) division**.

| N (array length) | N/2 (floor) | Element must appear |
|---|---|---|
| 8 | 4 | more than 4 times (i.e. ≥ 5) |
| 9 | 4 (floor of 4.5) | more than 4 times (i.e. ≥ 5) |
| 7 | 3 | more than 3 times (i.e. ≥ 4) |
| 6 | 3 | more than 3 times (i.e. ≥ 4) |

So for `N = 9`, you take the floor value of `9/2`, which is `4`, and the element must appear more than 4 times.

### Worked example from the lecture

```
arr[] = [2, 2, 3, 3, 1, 2, 2]
```

- Length of the array: count them — 1, 2, 3, 4, 5, 6, 7 → **N = 7**
- Threshold: element must appear more than `7/2 = 3` times.
- `2` appears **4 times** (positions 0, 1, 5, 6).
- `4 > 3` ✔
- **Answer = 2**

So the required output is simply: given an array, return the element that appears more than N/2 times.

### An important structural fact (used heavily later)

**At most one element can be a majority element.** If one element occupies more than half of the array, there is no room left for a second element to also occupy more than half. This property is the foundation of the optimal algorithm.

---

## 2. Approach 1 — Brute Force (O(N²))

### Idea

The most natural thing to do: **pick up one element at a time, and scan the whole array counting how many times that element occurs.** If that count ever exceeds `N/2`, that element is the answer.

For `arr = [2, 2, 3, 3, 1, 2, 2]`:
- First pick `2` (index 0) → scan the whole array → count = 4 → `4 > 3` → answer found.
- (If it hadn't matched, you would pick `2` at index 1, scan again; then `3` at index 2, scan again; then `3` at index 3, scan again; and so on.)

Yes, this repeats work (`2` is picked and re-scanned multiple times), but brute force is allowed to be wasteful — its job is to be obviously correct.

### Code (as written on the blackboard)

```cpp
for (i = 0; i < n; i++) {
    cnt = 0;                     // fresh count for every candidate
    for (j = 0; j < n; j++) {
        if (arr[j] == arr[i]) {  // compare candidate arr[i] with every element
            cnt++;
        }
    }
    if (cnt > n/2) return arr[i]; // majority condition satisfied
}
return -1;                        // no majority element exists
```

### Line-by-line logic

- **Outer loop (`i`)** — selects the candidate element `arr[i]`.
- **`cnt = 0` inside the outer loop** — the counter must be reset for every new candidate. Forgetting this reset is a classic bug: the counts of different elements would pile up together.
- **Inner loop (`j`)** — scans the entire array and increments `cnt` every time an element equal to the candidate is found.
- **`if (cnt > n/2) return arr[i];`** — the check is done *after* the inner scan finishes, i.e. once the candidate's total frequency is known. Note again: `>`, not `>=`.
- **`return -1;` after the loops** — reached only if no element crossed `N/2`. This is the "no majority element" case. (In some problem statements the majority element is guaranteed to exist; then this line is just a formality.)

### Complexity

| Metric | Value | Reason |
|---|---|---|
| Time | **O(N²)** | Two nested loops, each running N times |
| Space | **O(1)** | Only a counter variable is used |

### Interview reality check

If you give the O(N²) solution, **the interviewer will not be happy and will ask you to optimize.** That is your cue to move to the hashing solution.

---

## 3. Approach 2 — Better Solution using Hashing (Map)

### How to *derive* this in an interview (thought process)

This is the reasoning the instructor wants you to voice out loud:

1. The brute force takes `N²`. So the better solution has to be of an order strictly better than `N²` — something like `N log N`, `O(N)`, or `O(2N)`.
2. Ask yourself: *what am I actually doing in the brute force?* → **I am counting.** I am counting occurrences and checking who crosses `N/2`.
3. Whenever the task is "keep track of how many times something occurs", the technique that immediately comes to mind is **hashing**.

So: build a frequency map, then look for the key whose frequency exceeds `N/2`.

### Map structure

A hash map storing `(element, count)` — i.e. **element = key**, **count = value**:

```
(el, cnt)  ⟹  (key, value)
```

### Dry run on `arr[] = [2, 2, 3, 3, 1, 2, 2]`

Walk through the array once, incrementing the count of each element you see:

| Step | Element read | Map update |
|---|---|---|
| 1 | 2 | `(2, 1)` — "you occur once" |
| 2 | 2 | `(2, 2)` — "you occur twice" |
| 3 | 3 | `(3, 1)` |
| 4 | 3 | `(3, 2)` |
| 5 | 1 | `(1, 1)` |
| 6 | 2 | `(2, 3)` |
| 7 | 2 | `(2, 4)` |

**Final map contents:**

```
(1, 1)
(2, 4)
(3, 2)
```

Now **iterate over the map** (this is the second phase). Check each value against `N/2 = 3`:

- `1 → 1`, not > 3
- `2 → 4`, **4 > 3 ✔ → answer = 2**
- `3 → 2`, not > 3

So the answer is `2`.

### Code (CodeStudio, C++)

```cpp
#include <bits/stdc++.h>

int majorityElement(vector<int> v) {
    map<int, int> mpp;                        // element -> frequency

    // Phase 1: build the frequency map
    for (int i = 0; i < v.size(); i++) {
        mpp[v[i]]++;                          // "mark" this element in the map
    }

    // Phase 2: scan the map for a frequency > N/2
    for (auto it : mpp) {
        if (it.second > (v.size() / 2)) {     // it.second = count (value)
            return it.first;                  // it.first  = element (key)
        }
    }

    return -1;                                // no majority element
}
```

### Key points about the code

- `mpp[v[i]]++` — in C++, if the key does not exist yet, it is **default-constructed to 0** and then incremented to 1. So you never need an explicit "if key exists" check.
- `for (auto it : mpp)` — this is how you iterate a map in C++. Each `it` is a `(key, value)` pair:
  - `it.first` → the element (key)
  - `it.second` → the count (value)
- `it.second > v.size() / 2` — again the strict `>`.
- `return -1;` at the end handles the case where nobody crossed the threshold.
- **Compilation gotcha from the video:** the first run failed because `map` was not declared — the fix was adding `#include <bits/stdc++.h>`. After that it ran fine.
- Java and Python versions of this code are available in the video description (same logic with `HashMap` / `dict`).

### Complexity analysis (this is where interviewers dig)

**Time complexity:**

- First loop: runs `N` times → `O(N)`.
- But each insertion/lookup into a C++ `std::map` (an **ordered map**, implemented as a balanced BST) costs `O(log N)`.
  → so phase 1 is `O(N log N)`.
- If you use `unordered_map` instead, that `log N` factor disappears — **but only in the average and best case**. In the **worst case** (heavy hash collisions), `unordered_map` operations can degrade and end up taking `O(N)` time internally.
- Second loop: iterating over the map. How many elements can the map hold? If the array is something like `[1, 2, 3, 4, 5, 6]` (all unique), every element ends up as a separate key. So in the worst case the map holds `N` entries → `O(N)`.

**Total time = O(N log N) + O(N)** with `std::map`
(or **O(N) + O(N)** on average with `unordered_map`).

**Space complexity: O(N)** — you are storing elements in a map data structure. Remember: this worst case of `N` entries only happens when **the array contains all unique elements**.

### Interview reality check

The moment you present the hashing solution, the interviewer says:
> "Hey wait, you are using additional space. Can we please optimize this?"

That is your cue for the most optimal solution — **Moore's Voting Algorithm**.

---

## 4. Approach 3 — Optimal: Moore's Voting Algorithm

### A word of warning before the algorithm

- **You cannot invent this algorithm sitting in an interview.** You must know it beforehand.
- But that does **not** mean mug it up. If you rote-recite the algorithm line by line, the interviewer will immediately understand that you have memorised it.
- The interviewer grills you on **thought process and intuition**, not on the lines of code.
- So learn the *why* behind every step — that is exactly why the instructor teaches it as a dry run with reasoning attached to each step.

### The algorithm in one sentence

Maintain two variables — a candidate **`el`** (element) and a **`cnt`** (count). Sweep the array once: if `cnt == 0`, adopt the current element as the new candidate with `cnt = 1`; otherwise increment `cnt` if the current element equals the candidate, and decrement it if it does not.

### The crucial conceptual point: what does `cnt` mean?

> **`cnt` does NOT store how many times `el` appears.** It has a completely different significance.

`cnt` is a **running balance** between:
- occurrences of the candidate element (`+1` each), and
- occurrences of *everything else* (`−1` each).

So `cnt` measures **by how much the candidate is currently "winning"** inside the current section of the array. When `cnt` hits 0, the candidate has been exactly cancelled out by the other elements in that section.

### Full dry run

Array used in the lecture (**N = 16**):

```
arr[] = [7, 7, 5, 7, 5, 1, 5, 7, 5, 5, 7, 7, 5, 5, 5, 5]
index:   0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
```

Initially: `cnt = 0`, `el` = uninitialised (no element has been taken yet).

#### Segment 1 — indices 0…5: `[7, 7, 5, 7, 5, 1]`

| i | v[i] | Condition | Action | el | cnt |
|---|---|---|---|---|---|
| 0 | 7 | `cnt == 0` | take 7 as candidate | 7 | 0 → 1 |
| 1 | 7 | equals `el` | increment | 7 | 1 → 2 |
| 2 | 5 | ≠ `el` | decrement | 7 | 2 → 1 |
| 3 | 7 | equals `el` | increment | 7 | 1 → 2 |
| 4 | 5 | ≠ `el` | decrement | 7 | 2 → 1 |
| 5 | 1 | ≠ `el` | decrement | 7 | 1 → **0** |

**Reasoning at `cnt = 0`:** In this sub-array `[7, 7, 5, 7, 5, 1]`:
- `7` appears **3 times** → three `++` operations.
- "Other" elements (`5, 5, 1`) appear **3 times** → three `--` operations.
- `3 plus` and `3 minus` cancel exactly → `cnt = 0`.

**What can we conclude with certainty?** Considering only this 6-length sub-array, **7 is definitely not the majority element**, because:
- If `7` were the majority of these 6 elements, it would have to appear more than 3 times — say 4 times. To cancel 4 you would need 4 other elements, but only 2 slots remain. **You simply cannot cancel a true majority element.**
- `7` appeared exactly 3 times in a 6-length sub-array — that is **equal to N/2, not more than N/2** — so it got cancelled.

So up to index 5, 7 cannot be the answer, and we discard it and start fresh.

#### Segment 2 — indices 6…7: `[5, 7]`

| i | v[i] | Condition | Action | el | cnt |
|---|---|---|---|---|---|
| 6 | 5 | `cnt == 0` | take 5 as new candidate | 5 | 0 → 1 |
| 7 | 7 | ≠ `el` | decrement | 5 | 1 → **0** |

In the sub-array `[5, 7]`, `5` appeared once and `7` appeared once → they cancel → `5` cannot be the majority **of this section**.

#### Segment 3 — indices 8…11: `[5, 5, 7, 7]`

| i | v[i] | Condition | Action | el | cnt |
|---|---|---|---|---|---|
| 8 | 5 | `cnt == 0` | take 5 as new candidate | 5 | 0 → 1 |
| 9 | 5 | equals `el` | increment | 5 | 1 → 2 |
| 10 | 7 | ≠ `el` | decrement | 5 | 2 → 1 |
| 11 | 7 | ≠ `el` | decrement | 5 | 1 → **0** |

Here `5` occurs twice and `7` occurs twice → perfectly cancelled → `5` cannot be the majority of this section either.

**Observation so far:** every candidate we picked got cancelled out by the other elements. **Nobody has dominated yet.** We are still searching for someone who survives.

#### Segment 4 — indices 12…15: `[5, 5, 5, 5]`

| i | v[i] | Condition | Action | el | cnt |
|---|---|---|---|---|---|
| 12 | 5 | `cnt == 0` | take 5 as new candidate | 5 | 0 → 1 |
| 13 | 5 | equals `el` | increment | 5 | 1 → 2 |
| 14 | 5 | equals `el` | increment | 5 | 2 → 3 |
| 15 | 5 | equals `el` | increment | 5 | 3 → 4 |

**Array ends here** with `el = 5`, `cnt = 4`.

(Segment lengths check out: 6 + 2 + 4 + 4 = 16 = N.)

### The intuition, stated precisely

At the end of the sweep we are left with `el = 5`. What does that mean?

> **Every other candidate got cancelled out within its own section. If someone is still standing at the end — someone who did NOT get cancelled — that is the only possible majority element.**

And the core reason this works:

> **If an element appears more than N/2 times, it can never be fully cancelled out.** Every occurrence of a different element can kill at most one occurrence of the majority element, and there are fewer than N/2 "other" elements in total. So a true majority always survives with a positive balance.

### ⚠️ The essential caveat: `el` is only a *candidate*

The instructor stresses this repeatedly:

> **"IF there exists a majority element, it will be `5` and no one else. IF, IF!"**

The algorithm guarantees only this conditional statement. It does **not** guarantee that a majority element exists at all. If no majority exists, the algorithm still returns *some* element — a garbage candidate.

**Counter-example given in the lecture:** suppose the last portion of the array had been `[1, 1, 1, 1]` instead of `[5, 5, 5, 5]`. Then the algorithm would finish with `el = 1` and `cnt = 4`. But is `1` the majority? `1` would appear only about 5 times in a 16-length array, and you need **more than 8** occurrences to be the majority. So `1` would be a false candidate.

This is precisely why a **verification step is required**.

### The verification step

After Moore's voting produces `el`, simply **iterate over the array once more and count how many times `el` actually appears.** If that count is `> N/2`, return `el`; otherwise there is no majority element (return `-1` or whatever the problem asks).

For our array with `el = 5`:
- `5` occurs at indices 2, 4, 6, 8, 9, 12, 13, 14, 15 → **9 times**.
- `9 > 16/2 = 8` ✔ → **5 is the majority element.**

**A beautiful detail worth noticing:** `5` appears 9 times in total, of which only **4** survived uncancelled at the end. The other **5** occurrences of `5` were spent cancelling against non-5 elements during earlier segments. That is why `cnt` at the end (4) is *not* the frequency of the element (9) — it is only the surviving surplus.

### Also remember

> **Whatever the value of `cnt` is at the end, it does not represent anything meaningful.** Never return it or use it as a frequency.

### The algorithm as two clean steps

1. **Apply Moore's Voting Algorithm** → obtain the candidate element `el`.
2. **Verify** that `el` really appears more than `N/2` times.

### Code (CodeStudio, C++) — submitted successfully, 11/11 test cases passed

```cpp
#include <bits/stdc++.h>

int majorityElement(vector<int> v) {
    int cnt = 0;      // running balance, NOT a frequency
    int el;           // current candidate

    // ---------- Step 1: Moore's Voting ----------
    for (int i = 0; i < v.size(); i++) {
        if (cnt == 0) {
            cnt = 1;          // start a brand-new section
            el  = v[i];       // adopt current element as candidate
        }
        else if (v[i] == el) {
            cnt++;            // candidate reinforced
        }
        else {
            cnt--;            // candidate opposed / cancelled
        }
    }

    // ---------- Step 2: Verification ----------
    int cnt1 = 0;
    for (int i = 0; i < v.size(); i++) {
        if (v[i] == el) cnt1++;
    }

    if (cnt1 > (v.size() / 2)) {
        return el;
    }
    return -1;               // no majority element exists
}
```

### Execution flow explained (why just three branches suffice)

The loop body is only three cases, and they are mutually exclusive:

1. **`cnt == 0`** → "I am starting a check for a **new section**." Assign `cnt = 1`, `el = v[i]`, move on (`i++`).
2. **`v[i] == el`** → the candidate is confirmed again → `cnt++`, move on.
3. **otherwise** → the candidate is contradicted → `cnt--`, move on.

The elegance: when decrements drive `cnt` down to 0, the very next iteration hits case 1 again, **automatically replacing the candidate with a new element and beginning a fresh section**. There is no explicit "reset" logic needed — the `cnt == 0` check does it for you.

### Implementation notes / bugs to avoid

- The video initially failed to compile/run because `cnt1++` was missed inside the verification loop — a reminder to actually increment your verification counter.
- `el` is declared uninitialised. This is safe here **only because the first iteration always has `cnt == 0`** and therefore always assigns `el` (assuming the array is non-empty). For an empty array this would be undefined behaviour — mention guarding `n == 0` if the interviewer asks.
- You can store `v.size()` in a variable `n` instead of calling `v.size()` repeatedly — cleaner and slightly cheaper.
- Use `>` and not `>=` in the verification.

### Complexity

| Metric | Value | Reason |
|---|---|---|
| Time | **O(N) + O(N) = O(2N)** | One pass for voting, one pass for verification |
| Time (if majority is guaranteed) | **O(N)** | Verification pass can be skipped entirely |
| Space | **O(1)** | Only `el`, `cnt`, `cnt1` — no extra data structure |

**When to skip the verification pass:** if the problem statement explicitly says *a majority element always exists*, the second loop is unnecessary and should never be executed. You only perform verification when the array **might not** contain a majority element.

---

## 5. Comparison of All Three Approaches

| Approach | Core idea | Time | Space | Interviewer's reaction |
|---|---|---|---|---|
| Brute force | Pick each element, count via full scan | O(N²) | O(1) | "Optimize it." |
| Better (Hashing) | Build frequency map, scan map for count > N/2 | O(N log N) + O(N) with `map`; O(N) + O(N) avg with `unordered_map` | O(N) | "You're using extra space — optimize." |
| Optimal (Moore's Voting) | Cancel out pairs of different elements; survivor is the candidate; then verify | O(2N), or O(N) if majority guaranteed | **O(1)** | Accepted ✔ |

---

## 6. Key Takeaways / Revision Checklist

1. **"More than N/2", never "equal to N/2".** `N/2` uses floor division.
2. **At most one majority element can exist** — this is why cancellation works.
3. **Hashing is the natural "better" step** whenever the problem is about counting frequencies.
4. **`std::map` costs an extra `log N`; `unordered_map` removes it only in average/best case** — in the worst case `unordered_map` operations can degrade to `O(N)`.
5. **The map can hold up to N entries**, and that worst case occurs when all array elements are unique → `O(N)` space.
6. **In Moore's algorithm, `cnt` is a balance, not a frequency.** Its final value is meaningless.
7. **`cnt == 0` means "start a new section with a new candidate."**
8. **The algorithm only guarantees a conditional result:** *if* a majority element exists, it must be `el`. Hence the **verification pass is mandatory** unless the problem guarantees the majority's existence.
9. **The intuition in one line:** an element appearing more than N/2 times can never be fully cancelled, because there are not enough other elements to cancel it.
10. **Do not mug up Moore's algorithm** — be ready to explain the cancellation intuition and the section-wise dry run, because the interviewer grills the thought process, not the syntax.
