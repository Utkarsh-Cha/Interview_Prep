# Count the Number of Subarrays with Sum K

**Course context:** Strivers A2Z DSA Course → Step 3: *Solve Problems on Arrays* → Section 3.2 *Medium*.
This is the **last problem of the Medium-Arrays section**, so completing it wraps up the medium array section.

**Prerequisite:** This problem's optimal solution is *heavily* dependent on the **prefix-sum + hashmap** technique (taught earlier in the course, e.g. "Longest subarray with sum K" with positives and negatives). If you don't know prefix sum with hashing, learn that first — otherwise the optimal approach will feel like magic.

---

## 1. Problem Statement

Given an array `arr[]` and an integer `K`, **find the number of subarrays whose sum is exactly K**.

### What is a subarray?

> A subarray is **any contiguous part of the array**.

The word **contiguous** is the single most important word in the definition.

* It can be the **entire array**.
* It can be a **single element**.
* It can be 2 elements, 3 elements, any length — **but the elements must be taken consecutively, without skipping anything.**

**Example of what is NOT a subarray:**

For `arr[] = {1, 2, 3, -3, 1, 1, 1, 4, 2, -3}`:

* `{1, 2}` → ✅ subarray (indices 0–1, contiguous)
* `{3, -3, 1}` → ✅ subarray (indices 2–4, contiguous)
* `{1, 4, 2}` → ❌ **NOT a subarray**, because we skipped elements in between. This is a **subsequence**, not a subarray.

**Key distinction to remember:**

| Term | Requirement |
|---|---|
| **Subarray** | Contiguous — elements must be consecutive |
| **Subsequence** | Order preserved, but elements may be skipped |

---

## 2. Worked Example (Understanding the Answer)

```
arr[] = { 1, 2, 3, -3, 1, 1, 1, 4, 2, -3 }
index    0  1  2   3  4  5  6  7  8   9
K = 3
```

Let's list every contiguous stretch that sums to exactly 3:

| # | Subarray | Indices | Verification |
|---|---|---|---|
| 1 | `{1, 2}` | 0–1 | 1 + 2 = 3 |
| 2 | `{1, 2, 3, -3}` | 0–3 | 1 + 2 + 3 − 3 = 3 |
| 3 | `{3}` | 2–2 | 3 |
| 4 | `{2, 3, -3, 1}` | 1–4 | 2 + 3 − 3 + 1 = 3 |
| 5 | `{3, -3, 1, 1, 1}` | 2–6 | 3 − 3 + 1 + 1 + 1 = 3 |
| 6 | `{1, 1, 1}` | 4–6 | 1 + 1 + 1 = 3 |
| 7 | `{4, 2, -3}` | 7–9 | 4 + 2 − 3 = 3 |
| 8 | `{-3, 1, 1, 1, 4, 2, -3}` | 3–9 | −3 + 1 + 1 + 1 + 4 + 2 − 3 = 3 |

**Total = 8 subarrays.**

Notice how subarrays #2, #5 and #8 exploit the fact that `3` and `-3` cancel out — this is exactly why the array contains negative numbers, to stop you from assuming a simple two-pointer/sliding-window solution works.

---

## 3. Approach 1 — Brute Force (Generate All Subarrays)

### Intuition

In an interview, the very first solution you should state is the naive one: **generate every possible subarray, compute its sum, and check if it equals K.**

### How do we generate all subarrays?

Observe the pattern of subarray generation:

* Start at index 0: take `{arr[0]}`, then `{arr[0], arr[1]}`, then `{arr[0], arr[1], arr[2]}` … keep extending till the last index.
* Then start at index 1: take `{arr[1]}`, then `{arr[1], arr[2]}`, … till the last index.
* Then start at index 2: `{arr[2]}`, `{arr[2], arr[3]}`, … till the end.
* And so on.

So we need **two pointers**:

* `i` = the **starting index** of the subarray — moves from `0` to `n−1`.
* `j` = the **ending index** of the subarray — for each fixed `i`, `j` moves from `i` to `n−1`.

**Therefore, for any pair `(i, j)`, the subarray is exactly `arr[i … j]`.** This is the crucial observation.

### Computing the sum

Since we know the subarray is `arr[i…j]`, we run a **third loop** with variable `k` going from `i` to `j`, accumulating `sum = sum + arr[k]`.

After that inner loop finishes, `sum` holds the sum of the whole subarray `arr[i…j]`, so we check `if (sum == K) cnt++`.

### Pseudocode (as written on the board)

```cpp
cnt = 0;
for (i = 0; i < n; i++) {          // start index of subarray
    for (j = i; j < n; j++) {      // end index of subarray
        sum = 0;
        for (k = i; k <= j; k++)   // traverse the subarray arr[i..j]
            sum = sum + arr[k];

        if (sum == K)
            cnt++;
    }
}
return cnt;
```

> **Note on the board typo:** the outer loop was mistakenly written as `for (j = 0; ...)`. It must be `i` — the outer loop controls the **start** index, the inner loop controls the **end** index.

### Complexity

* **Time:** ~**O(N³)**.
  It is *not exactly* N³ — the innermost loop runs a different number of times for each `(i, j)` pair (sometimes 1 element, sometimes 5, sometimes the whole array) — but it is **of the order of N³**. Say exactly this to an interviewer: *"near about N³"*.
* **Space:** O(1).

The interviewer will **not** be satisfied with O(N³) and will ask you to optimize.

---

## 4. Approach 2 — Better (Remove the Third Loop)

### The observation that kills the inner loop

Fix `i` at some index. Now watch what happens as `j` moves:

* `j = i` → subarray is `{1}` → sum = 1
* `j = i+1` → subarray is `{1, 2}` → sum = 3
* `j = i+2` → subarray is `{1, 2, 3}` → sum = 6
* `j = i+3` → subarray is `{1, 2, 3, -3}` → sum = 3

Between one step and the next, **only ONE new element gets added at the end**. The previous elements are exactly the same.

So re-computing the whole sum from scratch every time is **pure waste**. Instead:

* Reset `sum = 0` **once per `i`** (just before the `j` loop starts).
* Every time `j` advances, simply do `sum += arr[j]` — add only the newly included element.

This is an **incremental / running sum**, and it removes the innermost loop entirely.

### Pseudocode (as written on the board)

```cpp
cnt = 0;
for (i = 0; i < n; i++) {
    sum = 0;                    // reset once for each new starting point
    for (j = i; j < n; j++) {
        sum += arr[j];          // only the new element is added
        if (sum == K)
            cnt++;
    }
}
return cnt;
```

### Complexity

* **Time:** ~**O(N²)** (again, "near about N²").
* **Space:** **O(1)** — no extra data structure is used.

The interviewer will still push for better. Time for the optimal solution.

---

## 5. Approach 3 — Optimal (Prefix Sum + Hashmap)

### 5.1 The core idea of prefix sum

A **prefix sum** at index `i` = sum of all elements from index `0` up to index `i`.

Take the example array and walk to index 5:

```
1 + 2 + 3 + (-3) + 1 + 1 = 5   ... and up to index 6 it is 6
```

Now suppose:

* The **total prefix sum up to the current index** is `x` (call it `S`).
* The sum of the **last few elements** (a subarray ending at the current index) is `K`.

Then the sum of the **remaining front portion** must be `x − K`. This is plain arithmetic — you don't add anything up, you just subtract:

```
[--------------- total prefix sum = S ---------------]
[----- prefix = S − K -----][--- subarray = K ---]
                                      ↑ ends at current index
```

**Concrete check from the lecture:** if the prefix sum up to some index is `6`, and the last three elements sum to `3`, then the front portion must sum to `6 − 3 = 3`. You never add those front elements — you deduce their sum.

### 5.2 The reverse-engineering trick (the heart of the solution)

Here's the mental flip that makes everything work.

We want: **how many subarrays ending at the current index have sum K?**

That is a "sum in the middle" question, and middle sums are not something we can store or look up cheaply.

But every such subarray corresponds **one-to-one** with a prefix of the array whose sum is `S − K`:

> If a subarray with sum `K` ends at the current index, then the portion before it is a prefix with sum `S − K`.
> Conversely, every earlier prefix with sum `S − K`, when removed, leaves behind a subarray ending at the current index with sum exactly `K`.

Therefore:

> **Number of subarrays with sum K ending at the current index = number of times the prefix sum `S − K` has occurred before.**

This is why the *count* matters and not just existence: if the value `S − K` occurred **twice** in the past, then there are **two** different earlier cut-points, hence **two** different subarrays ending here with sum `K`.

**Why look backwards instead of forwards?**
Because **prefix sums are trivially storable as we iterate**, whereas "the sum of some arbitrary middle segment" cannot be looked up directly. We convert an unanswerable question ("how many middle segments sum to K?") into an answerable one ("how many earlier prefixes summed to S − K?").

### 5.3 What data structures do we need?

We are asking: *"Did a prefix sum of value `S − K` happen in the past, and how many times?"*
A question about **past occurrences and their frequency** → **hashmap**.

| Thing | Purpose |
|---|---|
| `map<int,int> mpp` | key = a prefix sum value, value = how many times that prefix sum has occurred so far |
| `preSum` | running prefix sum up to the current index |
| `cnt` | running count of valid subarrays (the answer) |

### 5.4 Why we must insert `(0, 1)` before starting

Initialize the map with **`mpp[0] = 1`**.

Meaning: *"Before picking up any element, there exists one prefix (the empty prefix) whose sum is 0."*

This entry accounts for subarrays that **start at index 0** — i.e., where the "front portion to remove" is *nothing at all*. Without it, every subarray beginning at index 0 would be missed. (Full proof with the instructor's homework example is in §7.)

---

## 6. Complete Dry Run

```
arr[] = { 1, 2, 3, -3, 1, 1, 1, 4, 2, -3 },  K = 3
Initial:  preSum = 0, cnt = 0, mpp = { 0 : 1 }
```

The rule applied at every index:

1. `preSum += arr[i]`
2. `remove = preSum − K`
3. `cnt += mpp[remove]` (how many times that prefix has occurred before)
4. `mpp[preSum]++`

| Step | `arr[i]` | `preSum` | Looking for `preSum − K` | Found in map? | `cnt` after | Map after |
|---|---|---|---|---|---|---|
| 1 | `1` | 1 | `1 − 3 = −2` | ❌ not present | 0 | `{0:1, 1:1}` |
| 2 | `2` | 3 | `3 − 3 = 0` | ✅ 1 occurrence | **1** | `{0:1, 1:1, 3:1}` |
| 3 | `3` | 6 | `6 − 3 = 3` | ✅ 1 occurrence | **2** | `{0:1, 1:1, 3:1, 6:1}` |
| 4 | `-3` | 3 | `3 − 3 = 0` | ✅ 1 occurrence | **3** | `{0:1, 1:1, 3:2, 6:1}` |
| 5 | `1` | 4 | `4 − 3 = 1` | ✅ 1 occurrence | **4** | `{…, 3:2, 6:1, 4:1}` |
| 6 | `1` | 5 | `5 − 3 = 2` | ❌ not present | 4 | `{…, 4:1, 5:1}` |
| 7 | `1` | 6 | `6 − 3 = 3` | ✅ **2 occurrences** | **6** | `{…, 5:1, 6:2}` |
| 8 | `4` | 10 | `10 − 3 = 7` | ❌ not present | 6 | `{…, 6:2, 10:1}` |
| 9 | `2` | 12 | `12 − 3 = 9` | ❌ not present | 6 | `{…, 10:1, 12:1}` |
| 10 | `-3` | 9 | `9 − 3 = 6` | ✅ **2 occurrences** | **8** | `{…, 12:1, 9:1}` |

**Final answer: `cnt = 8`** ✅ — matches the 8 subarrays we listed manually.

### Step-by-step reasoning (what actually happened at each step)

* **Step 1 (preSum = 1):** We have some elements summing to 1. To leave behind a subarray of sum 3 ending here, we'd need to discard a front portion summing to `−2`. No such prefix has ever occurred, so no subarray ends here with sum 3.

* **Step 2 (preSum = 3):** We need to discard a front portion summing to `0` — i.e., **discard nothing**. The map has `0 → 1` (the initial entry!). So `cnt` becomes 1. This is the subarray `{1, 2}`. **This is precisely why storing `(0, 1)` matters.**

* **Step 3 (preSum = 6):** Discard a prefix of sum 3. It occurred once (after index 1). Removing `{1, 2}` leaves `{3}` → sum 3 ✅. `cnt = 2`.

* **Step 4 (preSum = 3 again):** Discard prefix of sum 0 → discard nothing → the whole `{1, 2, 3, -3}` sums to 3 ✅. `cnt = 3`.
  **Important map detail:** `3` already existed in the map with count 1, so we **don't create a new key** — we **increment it to `3 → 2`**. That stored count of 2 is what will pay off in Step 7.

* **Step 5 (preSum = 4):** Discard prefix of sum 1 → that's the single element `{1}` at index 0. Removing it leaves `{2, 3, -3, 1}` → sum 3 ✅. `cnt = 4`.

* **Step 6 (preSum = 5):** Need prefix of sum 2 → never occurred. Nothing added.

* **Step 7 (preSum = 6):** Need prefix of sum 3, and it has occurred **twice** (at index 1 and at index 3). Two different cut points → **two** subarrays ending here:
  * remove prefix ending at index 1 → `{3, -3, 1, 1, 1}` ✅
  * remove prefix ending at index 3 → `{1, 1, 1}` ✅

  So `cnt += 2 → 6`. **This is exactly why the map stores a count, not a boolean.** Also note `6` already existed, so we update it to `6 → 2` instead of re-inserting.

* **Steps 8 & 9 (preSum = 10, 12):** Need prefixes of 7 and 9 respectively — neither exists. Nothing added.

* **Step 10 (preSum = 9):** Need prefix of sum 6, which occurred **twice** (at index 2 and index 6). Two subarrays ending at the last index:
  * remove prefix ending at index 2 → `{-3, 1, 1, 1, 4, 2, -3}` ✅
  * remove prefix ending at index 6 → `{4, 2, -3}` ✅

  `cnt += 2 → 8`. Iteration ends; `cnt` holds the final answer.

---

## 7. Why Storing `(0, 1)` Is Critical — Homework Example Solved

The instructor gives this as a self-exercise: try `arr[] = {3, -3, 1, 1, 1}` with `K = 3` **without** the initial `(0,1)` entry and see what breaks.

**Correct answer (by hand):** `{3}`, `{3, -3, 1, 1, 1}`, `{1, 1, 1}` → **3 subarrays**.

### Run WITHOUT `mpp[0] = 1` (map starts empty)

| `arr[i]` | `preSum` | need `preSum−3` | found | `cnt` | map after |
|---|---|---|---|---|---|
| 3 | 3 | 0 | ❌ (map empty!) | 0 | `{3:1}` |
| −3 | 0 | −3 | ❌ | 0 | `{3:1, 0:1}` |
| 1 | 1 | −2 | ❌ | 0 | `{…, 1:1}` |
| 1 | 2 | −1 | ❌ | 0 | `{…, 2:1}` |
| 1 | 3 | 0 | ✅ 1 | 1 | `{3:2, …}` |

**Result: 1 — WRONG.** We lost `{3}` and `{3, -3, 1, 1, 1}`.

### Run WITH `mpp[0] = 1`

| `arr[i]` | `preSum` | need `preSum−3` | found | `cnt` | map after |
|---|---|---|---|---|---|
| — | 0 | — | — | 0 | `{0:1}` |
| 3 | 3 | 0 | ✅ 1 | 1 | `{0:1, 3:1}` |
| −3 | 0 | −3 | ❌ | 1 | `{0:2, 3:1}` |
| 1 | 1 | −2 | ❌ | 1 | `{…, 1:1}` |
| 1 | 2 | −1 | ❌ | 1 | `{…, 2:1}` |
| 1 | 3 | 0 | ✅ **2** | **3** | `{0:2, 3:2, …}` |

**Result: 3 — CORRECT.**

**Takeaway:** The `(0, 1)` entry represents the **empty prefix**. Every subarray that starts at index 0 needs "remove nothing" to be a legal option. Without it, all such subarrays are silently dropped.

---

## 8. Final Code (C++)

```cpp
int findAllSubarraysWithGivenSum(vector<int> &arr, int k) {
    map<int, int> mpp;        // prefix sum -> number of occurrences
    mpp[0] = 1;               // empty prefix: sum 0 occurs once

    int preSum = 0, cnt = 0;

    for (int i = 0; i < arr.size(); i++) {
        preSum += arr[i];             // 1. running prefix sum

        int remove = preSum - k;      // 2. the prefix we'd need to discard
        cnt += mpp[remove];           // 3. every past occurrence = one subarray

        mpp[preSum] += 1;             // 4. record current prefix sum
    }

    return cnt;
}
```

> *"So much of logic, but only four lines of code."*

### Line-by-line execution flow

1. **`mpp[0] = 1`** — seed the empty prefix (see §7).
2. **`preSum += arr[i]`** — extend the prefix sum to include the current index.
3. **`int remove = preSum - k`** — the value of the front chunk that must be excluded so that what remains (ending at `i`) sums to exactly `k`.
4. **`cnt += mpp[remove]`** — add the **frequency**, not 1. Each earlier occurrence is a distinct valid starting point.
5. **`mpp[preSum] += 1`** — register the current prefix sum so future indices can use it.

### ⚠️ Order matters: look up BEFORE inserting

`cnt += mpp[remove]` must come **before** `mpp[preSum] += 1`.

Reason: if you insert first, then when `k == 0` you would have `remove == preSum`, and the current prefix would count *itself* as a removable front portion — producing an empty subarray as a false positive. Always **query the past first, then record the present.**

### ⚠️ A note on `mpp[remove]` with `std::map`

Using `operator[]` on a missing key **inserts** that key with value `0`. The returned `0` is correct for the algorithm (no matching prefix → add nothing), so the answer is right, but it does bloat the map with junk keys. A cleaner variant is:

```cpp
if (mpp.find(remove) != mpp.end()) cnt += mpp[remove];
```

Mentioning this in an interview shows attention to detail.

---

## 9. Complexity Analysis

### Time Complexity

* We iterate the array **once** → **O(N)**.
* Each iteration does one map lookup + one map insert/update. The cost depends on which map you choose:

| Data structure | Lookup/insert cost | Overall time |
|---|---|---|
| `unordered_map` (average / best case) | O(1) | **O(N)** |
| `unordered_map` (worst case, hash collisions) | O(N) | **O(N²)** |
| `map` (ordered, red-black tree) | O(log N) | **O(N × log N)** |

**How to state this in an interview:** *"It's O(N) times the map operation cost. With an ordered map that's O(N log N); with an unordered_map it's O(N) on average, but O(N²) in the worst case due to collisions."* Being explicit about this trade-off is exactly what interviewers look for.

### Space Complexity

* **O(N)** — in the worst case every prefix sum is distinct, so the hashmap stores up to N entries.

**This is the best achievable complexity for this problem — we cannot do better than O(N).**

---

## 10. Summary of All Three Approaches

| Approach | Core idea | Time | Space |
|---|---|---|---|
| **Brute force** | Generate every subarray `(i, j)` and re-sum it with a third loop | ~O(N³) | O(1) |
| **Better** | Keep a running sum while `j` expands; drop the third loop | ~O(N²) | O(1) |
| **Optimal** | Prefix sum + hashmap; count past occurrences of `preSum − K` | O(N) avg (O(N log N) with ordered map) | O(N) |

---

## 11. Key Points to Remember (Interview Checklist)

1. **Subarray = contiguous.** Confirm this definition out loud before solving; `{1, 4, 2}` from our array is a *subsequence*, not a subarray.
2. **Always present the progression:** brute force → better → optimal. Don't jump straight to the optimal answer.
3. **The central identity:**
   > `#subarrays with sum K ending at index i` = `#times prefix sum (preSum − K) has occurred before index i`
4. **Store counts, not booleans,** in the hashmap. A prefix sum can repeat (e.g. `3` occurred twice, `6` occurred twice in our dry run), and each repetition yields another valid subarray.
5. **Always initialize `mpp[0] = 1`.** It handles subarrays starting at index 0. Skipping it silently produces wrong answers.
6. **Query before update.** Compute `cnt += mpp[preSum − k]` before doing `mpp[preSum]++`, otherwise `k = 0` breaks.
7. **Negative numbers are the reason** sliding-window / two-pointer doesn't work here — the running sum is not monotonic, so you can't shrink a window safely. Prefix-sum hashing handles negatives naturally.
8. **Why look backwards?** Because prefix sums can be stored cheaply as we go, while arbitrary "middle segment" sums cannot be looked up directly. The reverse-engineering converts the hard question into an easy one.
9. Be ready to discuss the **map vs unordered_map** complexity trade-off.
