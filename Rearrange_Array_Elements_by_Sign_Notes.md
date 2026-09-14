# Rearrange Array Elements by Sign — Complete Study Notes

**Course context:** Striver's A2Z DSA Course/Sheet → Step 3: *Solve Problems on Arrays* → Step 3.2 (Medium).
Previous problem in the sheet was **Stock Buy and Sell**; this problem is **"Rearrange the array in alternating positive and negative items."**

**Platforms used in the lecture:**
- **LeetCode 2149 — Rearrange Array Elements by Sign** → this is **Variety 1** (equal positives and negatives).
- **CodeStudio — Alternate Numbers** → this is **Variety 2** (positives and negatives may be unequal).

---

## 1. Problem Statement — Variety 1

You are given an array that contains an **equal number of positive and negative elements**.

- If the length of the array is `N`, then:
  - Number of positive elements = `N/2`
  - Number of negative elements = `N/2`
- Therefore **`N` is always an even number**. (If `N` were odd, you could not split it into two equal halves.)
- The elements can appear in **any random order** in the input: `+ + - - + -`, or `+ - - - + +`, or any other arrangement. The input order of signs is not fixed.
- Assumption used throughout: **the array contains no zeros** (every element is strictly positive or strictly negative). This matters later, because the code decides sign with a single `if (x < 0)` check.

### What "rearrange by sign" means

The output array must follow the strict alternating pattern, **starting with a positive**:

```
index :  0   1   2   3   4   5
sign  :  +   -   +   -   +   -
```

**Two more rules that are easy to miss (and are the whole difficulty of the problem):**

1. The **relative order of the positive numbers must be preserved** — the positives must appear in the same sequence in which they appeared in the input.
2. The **relative order of the negative numbers must be preserved** — likewise for negatives.

So you are not free to shuffle numbers around; you only interleave two ordered streams (the positive stream and the negative stream).

### Worked example from the blackboard

```
arr[] = [ 3,  1, -2, -5,  2, -4 ]
                 ↓  rearrange
arr[] = [ 3, -2,  1, -5,  2, -4 ]
```

Checking it against the rules:

- `N = 6` → 3 positives and 3 negatives. ✔ (equal, and `N` is even)
- Positives in the input, in order: `3, 1, 2`
- Positives in the output, in order: `3, 1, 2` → **order preserved** ✔
- Negatives in the input, in order: `-2, -5, -4`
- Negatives in the output, in order: `-2, -5, -4` → **order preserved** ✔
- Signs in the output: `+ - + - + -` → **alternating, starting with positive** ✔

---

## 2. The Key Observation (the heart of both solutions)

Look at where each element lands in the answer:

```
value  :  3   -2    1   -5    2   -4
index  :  0    1    2    3    4    5
sign   :  +    -    +    -    +    -
```

- **All positive elements sit at EVEN indexes:** 0, 2, 4
- **All negative elements sit at ODD indexes:** 1, 3, 5

Now go one step further and number the positives themselves:

| Positive number | Its position among positives (i) | Index it must occupy |
|---|---|---|
| 3 | 0 | 0 |
| 1 | 1 | 2 |
| 2 | 2 | 4 |
| (if a 4th existed) | 3 | 6 |

The pattern is exact: **the i-th positive element goes to index `2*i`.**

Same reasoning for negatives:

| Negative number | Its position among negatives (i) | Index it must occupy |
|---|---|---|
| -2 | 0 | 1 |
| -5 | 1 | 3 |
| -4 | 2 | 5 |

**The i-th negative element goes to index `2*i + 1`.**

These two formulas — `2*i` and `2*i + 1` — are used by *every* solution in this lecture, including the follow-up variety. Memorise the derivation, not just the formula.

---

## 3. Brute Force Solution (Variety 1)

> **Interview advice from the lecture:** whenever this question is asked, **always state the brute force first**. It is the first solution that strikes anyone's head, and stating it shows the interviewer your thought process before you optimise.

### Idea

1. Take two empty containers:
   - `pos → []` — will hold all positive elements, in their original relative order.
   - `neg → []` — will hold all negative elements, in their original relative order.
   - You *may* use fixed arrays of size `N/2` each (you know the sizes in advance for Variety 1), but a **list/vector is preferred** because it is simpler and generalises better.
2. **Pass 1 — collect:** iterate over the whole input array. Every positive element is pushed into `pos`, every negative element is pushed into `neg`.
3. **Pass 2 — place back:** use the `2*i` / `2*i + 1` formulas to write them into the answer.

### Pass 1 dry run on `arr[] = [3, 1, -2, -5, 2, -4]`

| Step | Element | Sign | Action |
|---|---|---|---|
| 1 | 3 | + | `pos → [3]` |
| 2 | 1 | + | `pos → [3, 1]` |
| 3 | -2 | − | `neg → [-2]` |
| 4 | -5 | − | `neg → [-2, -5]` |
| 5 | 2 | + | `pos → [3, 1, 2]` |
| 6 | -4 | − | `neg → [-2, -5, -4]` |

After this single iteration we have:

```
pos = [ 3,  1,  2]     ← positives, in correct relative order
neg = [-2, -5, -4]     ← negatives, in correct relative order
```

Because we pushed elements in the order we met them, **the relative order is automatically preserved** — this is exactly why the problem's ordering constraint is satisfied for free.

### Pass 2 — pseudo-code written on the board

```cpp
for (i = 0; i < N/2; i++) {
    arr[2 * i]     = pos[i];   // even index gets the i-th positive
    arr[2 * i + 1] = neg[i];   // odd  index gets the i-th negative
}
```

Why does the loop run only `N/2` times? Because there are exactly `N/2` positives and `N/2` negatives, and each iteration places **one positive and one negative** — i.e. two elements per iteration, `2 × N/2 = N` elements total. 

### Pass 2 dry run

| i | `arr[2*i] = pos[i]` | `arr[2*i+1] = neg[i]` | Array so far |
|---|---|---|---|
| 0 | `arr[0] = 3` | `arr[1] = -2` | `[3, -2, _, _, _, _]` |
| 1 | `arr[2] = 1` | `arr[3] = -5` | `[3, -2, 1, -5, _, _]` |
| 2 | `arr[4] = 2` | `arr[5] = -4` | `[3, -2, 1, -5, 2, -4]` ✔ |

### Complexity — Brute Force

- **Time:** `O(N)` for the collection pass **+** `O(N/2)` for the placement pass = **`O(N + N/2) = O(1.5N)`**, i.e. `O(N)` asymptotically.
  - ⚠️ **Correction made at the end of the lecture:** on the board this was first written as `O(2N)`. That was a **typo**. The second loop runs only `N/2` times (not `N`), so the correct count is `O(N + N/2)`.
- **Space:** **`O(N)`** — the two extra containers hold `N/2 + N/2 = N` elements.

### Why the interviewer will not be satisfied

Two things stand out:
1. It takes **multiple passes** (collect, then place).
2. It uses **extra space**.

**Can the extra space be removed?** Realistically, **no** — you have to store something somewhere to reorder the elements correctly, so some extra space is unavoidable in this approach family.

**Can we reduce two passes into one?** **Yes** — and that is exactly where the optimal solution comes in.

---

## 4. Optimal Solution (Variety 1) — Single Pass

### Idea

Instead of first bucketing into `pos`/`neg` and then writing back, we write **directly into an answer array during the one and only pass**, using two moving index pointers.

- Create `ans[]` of size `N`.
- Keep `posIndex = 0` → the **next free even slot** (where the next positive must go).
- Keep `negIndex = 1` → the **next free odd slot** (where the next negative must go).
- Walk the input array once:
  - If the current element is **positive** → place it at `ans[posIndex]`, then advance `posIndex += 2` (the next even slot).
  - If the current element is **negative** → place it at `ans[negIndex]`, then advance `negIndex += 2` (the next odd slot).

The intuition from the lecture, phrased as a conversation with the element: *"You're a positive — go to the first place reserved for a positive. Once you've taken it, the next positive's first available place is two steps ahead."* Since positives only ever occupy even slots and negatives only odd slots, the two pointers never collide, and each just marches forward in steps of 2.

Because we consume the input strictly left to right and each pointer only moves forward, **relative order is again preserved automatically**.

### Full dry run on `arr[] = [3, 1, -2, -5, 2, -4]`

Start: `ans = [_, _, _, _, _, _]`, `posIndex = 0`, `negIndex = 1`.

| Element | Sign | Placed at | `ans` after step | posIndex | negIndex |
|---|---|---|---|---|---|
| 3 | + | `ans[0] = 3` | `[3, _, _, _, _, _]` | 0 → 2 | 1 |
| 1 | + | `ans[2] = 1` | `[3, _, 1, _, _, _]` | 2 → 4 | 1 |
| -2 | − | `ans[1] = -2` | `[3, -2, 1, _, _, _]` | 4 | 1 → 3 |
| -5 | − | `ans[3] = -5` | `[3, -2, 1, -5, _, _]` | 4 | 3 → 5 |
| 2 | + | `ans[4] = 2` | `[3, -2, 1, -5, 2, _]` | 4 → 6 | 5 |
| -4 | − | `ans[5] = -4` | `[3, -2, 1, -5, 2, -4]` | 6 | 5 → 7 |

Final: `ans[] = [3, -2, 1, -5, 2, -4]` ✔

Note that at the end `posIndex = 6` and `negIndex = 7` — both have run past the array. That is harmless because the loop has finished; no further write happens. This is worth knowing so you don't panic about out-of-bounds: **the pointers are advanced after the last write but never used again.**

### Important honesty point made in the lecture

We are **not** rearranging the original array in place. We build a **new answer array** and return that. This is accepted because the problem asks you to *return* the rearranged array.

### Complexity — Optimal

- **Time:** **`O(N)`** — exactly one pass over the input.
- **Space:** **`O(N)`** — for the answer array. Note that this space is used **to return the answer**, not as auxiliary scratch space, so it is not really "wasted" space in the same way the brute force's `pos`/`neg` buckets were.

### How big is the improvement, really?

The lecture is explicit and honest about this: **it is not a huge improvement**. Brute force is `O(1.5N)` time; optimal is `O(N)` time. Asymptotically both are `O(N)`. The gain is by a **very slight margin** — we saved the second pass and the two auxiliary buckets. Say this honestly in an interview rather than overselling it.

| | Brute Force | Optimal |
|---|---|---|
| Passes | 2 (collect + place) | 1 |
| Time | `O(N + N/2) = O(1.5N)` | `O(N)` |
| Extra space | `O(N)` (pos + neg buckets) | `O(N)` (answer array only) |

---

## 5. Code — Variety 1 (LeetCode 2149)

```cpp
class Solution {
public:
    vector<int> rearrangeArray(vector<int>& nums) {
        int n = nums.size();
        vector<int> ans(n, 0);              // answer array of size n, filled with 0s

        int posIndex = 0, negIndex = 1;     // first even slot, first odd slot

        for (int i = 0; i < n; i++) {
            if (nums[i] < 0) {              // negative element
                ans[negIndex] = nums[i];
                negIndex += 2;              // next free odd slot
            }
            else {                          // positive element
                ans[posIndex] = nums[i];
                posIndex += 2;              // next free even slot
            }
        }
        return ans;
    }
};
```

### Line-by-line reasoning

- `int n = nums.size();` — we need the size both to allocate `ans` and to bound the loop.
- `vector<int> ans(n, 0);` — allocate the result up front with the exact size `n`. Pre-allocating matters because we write to arbitrary indexes (`posIndex`, `negIndex`) rather than pushing to the back, so the slots must already exist.
- `int posIndex = 0, negIndex = 1;` — the starting even and odd positions. This encodes the rule "**the answer starts with a positive**."
- `if (nums[i] < 0)` — a single comparison decides the sign. **This is only safe because the problem guarantees no zeros.** If zeros were possible, a `0` would fall into the `else` branch and be treated as positive.
- `negIndex += 2;` / `posIndex += 2;` — stepping by 2 keeps each pointer inside its own parity lane (odd stays odd, even stays even), so the alternating pattern is guaranteed by construction.
- `return ans;` — we return the new array, leaving `nums` untouched.

**Execution flow summary:** one linear scan; each element is classified once and written once; O(1) work per element.

**Submission result shown in the lecture:** Accepted — Runtime 237 ms, beats 59.79%.

---

## 6. Variety 2 — Unequal Positives and Negatives (CodeStudio: *Alternate Numbers*)

### Problem statement (as shown on screen)

> You are given an array `A` of size `N` with positive and negative numbers. Without altering the relative order of positive and negative numbers, you must return an array of alternate positive and negative values.
>
> **Note:** Start the array with a positive number. **If any of the positive and negative numbers are left, add them at the end without altering the order.**

The critical difference: **there is no guarantee that the count of positives equals the count of negatives.**

So there are exactly **two possible cases**:
1. `count(positives) > count(negatives)`
2. `count(negatives) > count(positives)`

(and the equality case, which we already solved, is absorbed into the code as shown later).

### What the answer should look like

**Example 1:** `arr[] = [1, 2, -4, -5, 3, 6]`
- Negatives: `-4, -5` → 2 of them
- Positives: `1, 2, 3, 6` → 4 of them
- So positives are in excess.

Build the answer:
1. Put a positive → `1`
2. Put a negative → `-4`
3. Put the next positive → `2`
4. Put the next negative → `-5`
5. Negatives are exhausted. **Whatever positives remain get appended at the end, in their original relative order** → `3`, then `6`.

```
ans[] = [1, -4, 2, -5, 3, 6]
```

**Example 2 (also on the board):** `arr[] = [-1, 2, 3, 4, -3, 1]`
- `pos = [2, 3, 4, 1]` (4 positives, order preserved)
- `neg = [-1, -3]` (2 negatives, order preserved)
- Number of negatives = 2, so `2 × 2 = 4` positions (indexes 0, 1, 2, 3) can be filled in the strict alternating pattern:

```
ans[] = [2, -1, 3, -3, _, _]
          0   1  2   3
```
- Remaining positives `[4, 1]` are appended from index 4 onwards:

```
ans[] = [2, -1, 3, -3, 4, 1]
```

### Why the optimal one-pass solution BREAKS here

The single-pass `posIndex/negIndex` trick **relies on the counts being equal**. It assumes that every even slot will get a positive and every odd slot will get a negative, forever. The moment one sign runs out, the remaining elements of the other sign must be packed **contiguously** at the end — not every second slot. The two-pointer parity scheme cannot express that, and `posIndex` would run past the end of the array.

So: **we deliberately fall back to the brute force solution.**

> **Big interview insight from the lecture:** This is an *amazing follow-up question* precisely because many candidates panic here. They assume the follow-up must need some clever new technique, they are not confident in their earlier (brute force) solution, and so they fail to fall back to it. **The right move is to confidently go back to the brute force approach and extend it.** Knowing *when a previous, "worse" solution is actually the right base* is itself a skill interviewers test.

### Algorithm for Variety 2

1. **Pass 1 — separate:** iterate the whole array; collect positives into `pos` and negatives into `neg`, preserving order.
2. **Fill the alternating block:** let `m = min(pos.size(), neg.size())`. The first `2*m` positions of the answer can be filled with the strict alternating pattern, using the same formulas:
   ```
   a[2*i]     = pos[i]
   a[2*i + 1] = neg[i]     for i = 0 .. m-1
   ```
   This is valid because for every one of those `m` pairs, both a positive and a negative are guaranteed to exist.
3. **Append the leftovers:** the majority sign has extra elements starting at index `m` inside its own bucket. They must be written into the answer starting at index `index = m * 2` (because the first `2*m` slots are already occupied), and then placed **consecutively** (`index++`, *not* `index += 2`).

**Where do the leftovers start — derivation:**
- In Example 2, `m = neg.size() = 2`. The first `2` positives and `2` negatives filled indexes `0, 1, 2, 3` → that's `2*2 = 4` slots.
- So the leftover writing begins at index `4 = neg.size() * 2`.
- And inside the `pos` bucket, positives `0` and `1` are already used, so the leftovers begin at bucket index `2 = neg.size()`.

### Pseudo-code written on the board (for the `pos > neg` case with 2 negatives)

```cpp
// Fill the alternating part (i < min(pos.size(), neg.size()), here 2)
for (i = 0; i < 2; i++) {
    arr[i * 2]     = pos[i];
    arr[i * 2 + 1] = neg[i];
}

// Fill remaining positives starting at index 4 (= neg.size() * 2)
ind = 4;
for (i = 2; i < pos.size(); i++) {   // i starts at neg.size()
    arr[ind] = pos[i];
    ind++;                            // consecutive, NOT +=2
}
```

---

## 7. Code — Variety 2 (CodeStudio: Alternate Numbers)

```cpp
vector<int> alternateNumbers(vector<int>& a) {
    vector<int> pos, neg;
    int n = a.size();

    // ---------- Step 1: separate positives and negatives ----------
    for (int i = 0; i < n; i++) {
        if (a[i] > 0) {
            pos.push_back(a[i]);
        }
        else {
            neg.push_back(a[i]);
        }
    }

    // ---------- Step 2: case A — more positives than negatives ----------
    if (pos.size() > neg.size()) {

        // alternating block: limited by the SMALLER count (negatives)
        for (int i = 0; i < neg.size(); i++) {
            a[2 * i]     = pos[i];
            a[2 * i + 1] = neg[i];
        }

        // leftovers: the extra positives, appended consecutively
        int index = neg.size() * 2;
        for (int i = neg.size(); i < pos.size(); i++) {
            a[index] = pos[i];
            index++;
        }
    }
    // ---------- Step 3: case B — more negatives (also covers equal) ----------
    else {
        for (int i = 0; i < pos.size(); i++) {
            a[2 * i]     = pos[i];
            a[2 * i + 1] = neg[i];
        }

        int index = pos.size() * 2;
        for (int i = pos.size(); i < neg.size(); i++) {
            a[index] = neg[i];
            index++;
        }
    }

    return a;   // modified in place, so return the same array
}
```

### Explanation of the important lines

- `if (a[i] > 0) pos.push_back(...) else neg.push_back(...)`
  Pushing in scan order is what preserves the **relative order** inside each bucket. (Again, no-zeros assumption: a `0` would be classified as negative here.)

- `if (pos.size() > neg.size())` … `else` …
  Two symmetric branches. **The `else` branch also handles the `pos.size() == neg.size()` case**, and that is deliberate.

- **Alternating loop bound = the smaller count.**
  In the `pos > neg` branch we loop `i < neg.size()`; in the other branch we loop `i < pos.size()`. Reason: you can only form a `(+, −)` pair as long as **both** buckets still have an element. The smaller bucket is the binding constraint.

- `int index = neg.size() * 2;` (or `pos.size() * 2;`)
  The number of slots already consumed by the alternating block. Derived earlier: `m` pairs × 2 elements per pair.

- `for (int i = neg.size(); i < pos.size(); i++)`
  Start reading leftovers from the first *unused* index of the larger bucket, which is exactly the size of the smaller bucket.

- `index++` (not `index += 2`)
  Leftovers are appended **back to back** at the end, because there is nothing of the other sign left to interleave with.

- **What happens when `pos.size() == neg.size()`?**
  Control goes to the `else`. The first loop runs `pos.size()` times and fills the entire array perfectly. Then `index = pos.size() * 2 = n`, and the leftover loop's condition `i < neg.size()` is immediately false since `i` starts at `pos.size() == neg.size()`. **The leftover loop never executes** — it is "done and dusted" by the first loop. So the equal case is handled correctly for free.

- `return a;`
  Unlike Variety 1, here we wrote the results **back into the original array `a`**, so we return `a` itself. (This is safe because the values were already safely copied out into `pos` and `neg` before any overwriting began — an important subtlety: you must not overwrite `a` before you have finished reading it, and Step 1 guarantees that.)

**Result shown in the lecture:** all 5/5 test cases passed on the first run.

### Dry run of the Variety 2 code on `a = [-1, 2, 3, 4, -3, 1]`

**Step 1:** `pos = [2, 3, 4, 1]`, `neg = [-1, -3]`

**Step 2:** `pos.size() = 4 > neg.size() = 2` → first branch.

Alternating loop (`i = 0 .. 1`):

| i | `a[2i] = pos[i]` | `a[2i+1] = neg[i]` | `a` so far |
|---|---|---|---|
| 0 | `a[0] = 2` | `a[1] = -1` | `[2, -1, 3, 4, -3, 1]` |
| 1 | `a[2] = 3` | `a[3] = -3` | `[2, -1, 3, -3, -3, 1]` |

Leftover loop: `index = 2 * 2 = 4`, `i` runs from `2` to `3`:

| i | `pos[i]` | Write | `a` so far |
|---|---|---|---|
| 2 | 4 | `a[4] = 4`, `index → 5` | `[2, -1, 3, -3, 4, 1]` |
| 3 | 1 | `a[5] = 1`, `index → 6` | `[2, -1, 3, -3, 4, 1]` |

**Final answer:** `[2, -1, 3, -3, 4, 1]` ✔

---

## 8. Complexity Analysis — Variety 2

The formula written on the board:

$$TC \;\rightarrow\; O(N) \;+\; O(\min(\text{pos}, \text{neg})) \;+\; O(\text{Leftovers})$$

Breaking it down:
- `O(N)` → the first pass that separates positives and negatives.
- `O(min(pos, neg))` → the alternating loop, which runs as many times as the **smaller** bucket's size.
- `O(Leftovers)` → the final loop over the surplus elements.

### Why these two terms trade off against each other

Notice that `min(pos, neg)` and `Leftovers` are **inversely related** — when one is large the other is small. Examine the extremes:

**Case 1 — everything is positive (or everything is negative):**
- `min(pos, neg) = 0` → the alternating loop does **zero** work.
- But then **all `N` elements are leftovers** → the leftover loop runs `N` times.
- Total: `O(N) + O(0) + O(N) = O(2N)`.

**Case 2 — equally divided (`N/2` positives, `N/2` negatives):**
- `min(pos, neg) = N/2` → the alternating loop runs `N/2` times.
- Leftovers = 0 → the leftover loop does nothing.
- Total: `O(N) + O(N/2) + O(0) = O(1.5N)`.

### Conclusion

- **Worst-case Time Complexity: `O(2N)`** — attained when the array is entirely positive or entirely negative.
- **Space Complexity: `O(N)`** — because we store every element across the two vectors `pos` and `neg` (`|pos| + |neg| = N`).

---

## 9. Consolidated Summary Table

| | Variety 1 — Brute Force | Variety 1 — Optimal | Variety 2 |
|---|---|---|---|
| Precondition | `#pos == #neg == N/2`, `N` even | `#pos == #neg == N/2`, `N` even | counts may be unequal |
| Approach | Bucket into `pos`/`neg`, then place with `2i` / `2i+1` | One pass, two parity pointers `posIndex=0`, `negIndex=1`, step `+2` | Bucket into `pos`/`neg`; alternate for `min(|pos|,|neg|)` pairs; append leftovers |
| Passes | 2 | 1 | 2 (separate + fill) |
| Time | `O(N + N/2) = O(1.5N)` | `O(N)` | `O(2N)` worst case |
| Space | `O(N)` | `O(N)` (answer array) | `O(N)` |
| Writes to | new array | new `ans` array | the original array `a`, in place |

---

## 10. Interview-Relevant Takeaways & Warnings

1. **Always present the brute force first.** It demonstrates your thought process and gives the interviewer something to ask you to optimise.
2. **State the key observation explicitly** — positives at even indexes, negatives at odd indexes, with the `i → 2i` and `i → 2i+1` mapping. This single observation powers every solution here.
3. **Be honest about the gain.** Optimal is `O(N)` vs brute's `O(1.5N)` — a small margin, and space stays `O(N)` in both. Don't oversell it.
4. **Extra space cannot realistically be removed** in this approach family — you need somewhere to build the correctly ordered result.
5. **The big follow-up trap:** when told positives ≠ negatives, many candidates freeze looking for a brand-new clever trick. The correct answer is to **fall back to the brute force and extend it**. Have the confidence to reuse your own earlier solution.
6. **The alternating loop must be bounded by the smaller count**, never by `N/2` and never by the larger count — otherwise you index out of bounds on the smaller bucket.
7. **Leftovers are appended consecutively** (`index++`), not at every second position (`index += 2`). Mixing this up is a classic bug.
8. **The equality case is automatically handled** by the `else` branch — the leftover loop simply never runs. You do not need a third branch.
9. **Relative order is preserved for free** in all approaches, because elements are always consumed left-to-right and pointers only move forward. Be able to say *why*, not just *that*.
10. **Zeros:** all the code assumes no zeros. If an interviewer adds zeros to the input, ask how a `0` should be classified — a single `< 0` or `> 0` test silently buckets it with one side.
11. **Order of operations matters in Variety 2:** you can only write back into the original array *after* you have finished copying everything into `pos` and `neg`; otherwise you would overwrite values you still need to read.
12. **Typo correction to remember:** the brute force for Variety 1 is `O(N + N/2)`, not `O(2N)`. The `O(2N)` figure belongs to Variety 2's worst case.
