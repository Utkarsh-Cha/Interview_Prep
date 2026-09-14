# Subarray Sums Divisible by K — Complete Study Notes
**LeetCode 974 · Difficulty: Medium · Array Playlist Video #34**
**Company Tags:** Microsoft · Paytm · Amazon

---

## 1. Problem Statement

You are given an integer array `nums` and an integer `K`. You must return **how many subarrays** of `nums` have a sum that is **divisible by K**.

A subarray is a *contiguous* portion of the array. The condition to satisfy is:

```
sum(subarray) % K == 0
```

### Worked Example from the Lecture

```
Input  : nums = [4, 5, 0, -2, -3, 1],  K = 5
Output : 7
```

The instructor enumerates all 7 qualifying subarrays on screen, one by one, using different colours:

| # | Subarray | Sum | Divisible by 5? |
|---|----------|-----|-----------------|
| 1 | `[4, 5, 0, -2, -3, 1]` (the whole array) | 4+5=9, +0=9, −2=7, −3=4, +1=**5** | ✔ |
| 2 | `[5]` | 5 | ✔ |
| 3 | `[5, 0]` | 5 | ✔ |
| 4 | `[0]` | 0 | ✔ (0 % 5 == 0) |
| 5 | `[5, 0, -2, -3]` | 0 | ✔ |
| 6 | `[0, -2, -3]` | −5 | ✔ |
| 7 | `[-2, -3]` | −5 | ✔ |

**Total = 7.**

### Important observations baked into this example
- **The entire array itself counts as a subarray.** The instructor explicitly points this out — don't forget the full array while enumerating.
- **A single element is also a subarray** (`[5]`, `[0]`).
- **`0` is divisible by every K** — `0 % 5 == 0`. So a subarray summing to 0 always counts.
- **Negative sums also count.** `−5 % 5 == 0`, so `[0, -2, -3]` and `[-2, -3]` are both valid. This foreshadows the negative-number edge case that becomes critical later.

---

## 2. Approach 1 — Brute Force, O(n³)

### Intuition

We simply generate **every possible subarray**, compute its sum, and check divisibility.

How do we generate every subarray? Fix a left boundary `i`, and let a right boundary `j` sweep from `i` all the way to the end of the array.

The instructor walks this through on a 6-cell diagram (indices `0 1 2 3 4 5`):

- Fix `i` at index 0. Start `j` at index 0.
  - Compute sum of `i..j` → check divisibility.
  - Move `j` to 1 → compute sum of `i..j` → check.
  - Move `j` to 2, 3, 4, 5 → each time compute sum of `i..j` and check.
- Once `j` has reached the end, **increment `i`** to index 1.
  - Now `j` restarts from `i` (index 1) and sweeps to the end again.
- Repeat until `i` reaches the end.

This covers every `(start, end)` pair exactly once, i.e. every subarray.

### Pseudocode (as written on screen)

```cpp
// Brute Force

result = 0;
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        // compute sum from i to j
        // check if divisible by K
        // if yes -> result++
    }
}
return result;
```

Fleshed out:

```cpp
int result = 0;
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        int Sum = 0;
        for (int t = i; t <= j; t++)   // <-- this inner loop is O(n)
            Sum += nums[t];
        if (Sum % K == 0)
            result++;
    }
}
return result;
```

### Why O(n³)?

Three nested O(n) costs stack up:

| Piece | Cost |
|-------|------|
| Outer loop on `i` | O(n) |
| Inner loop on `j` | O(n) |
| Recomputing the sum from `i` to `j` | O(n) in the worst case |
| **Total** | **O(n³)** |

- **Space:** O(1)

### Verdict and interview advice

- This will almost certainly give **Time Limit Exceeded** on LeetCode because the constraints are high.
- **Still write/mention the brute force.** The instructor stresses this: *"brute force likhna hamesha chahiye"* — you should always be able to state the brute force. It is the baseline you improve from, and interviewers explicitly want to see that progression.

---

## 3. Approach 2 — Prefix (Cumulative) Sum, O(n²)

### The single idea that removes one factor of `n`

Look again at the brute force. The `i` loop is O(n) and the `j` loop is O(n) — those are unavoidable if we insist on visiting every subarray. But the **third** O(n) — recomputing the sum from `i` to `j` — is pure waste. We keep re-adding the same numbers over and over.

**Claim:** the sum of any range `i..j` can be obtained in **O(1)** if we first convert the array into a **cumulative (prefix) sum array**.

### Building the cumulative sum array

Take the on-screen example array:

```
Index :  0   1   2   3   4   5
Value :  3   2   1   0   8   2
```

Replace each element by the running total of everything up to and including it:

```
3
3 + 2  = 5
5 + 1  = 6
6 + 0  = 6
6 + 8  = 14
14 + 2 = 16
```

Cumulative array:

```
Index :  0   1   2   3   4   5
nums  :  3   5   6   6  14  16
```

So `nums[k]` now means *"sum of the original elements from index 0 to index k"*.

### The range-sum formula

```
sum of original elements from i to j  =  nums[j] - nums[i - 1]
```

**Why it works:** `nums[j]` is everything from 0 to `j`. `nums[i-1]` is everything from 0 to `i-1` — i.e. exactly the prefix we want to discard. Subtracting removes it and leaves `i..j`.

**Verification from the lecture (i = 1, j = 4):**

- Direct computation on the original values: `2 + 1 + 0 + 8 = 11`
- Formula: `nums[4] − nums[1−1]` = `nums[4] − nums[0]` = `14 − 3` = **11** ✔

### The `i == 0` edge case

If `i = 0`, then `i - 1 = -1`, and `nums[-1]` is an **out-of-bounds access** — a real bug, not a theoretical one.

But think about what `i = 0` means: we want the sum from the very start of the array, so there is **no prefix to subtract**. The prefix sum `nums[j]` *is* the answer.

Check with `i = 0, j = 4`: direct sum `3 + 2 + 1 + 0 + 8 = 14`, and `nums[4] = 14` ✔

So guard it:

```cpp
if (i == 0) sum = nums[j];
else        sum = nums[j] - nums[i - 1];
```

Or, more compactly, using the **ternary operator** (the instructor notes this is just a "smart version" of the same `if/else`, taught in college):

```cpp
int sum = (i == 0) ? nums[j] : nums[j] - nums[i - 1];
```

### Approach-2 pseudocode (as shown on screen)

```cpp
// Approach-2 :- O(n^2)

int result = 0;
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        Sum = (i == 0) ? nums[j] : nums[j] - nums[i - 1];   // ---> O(1)
        if (Sum % K == 0)
            result++;
    }
}
```

### Full LeetCode implementation

```cpp
class Solution {
public:
    int subarraysDivByK(vector<int>& nums, int k) {
        int n = nums.size();
        int result = 0;

        // Step 1: convert nums into a cumulative (prefix) sum array, in-place
        for (int i = 1; i < n; i++) {
            nums[i] += nums[i - 1];
        }

        // Step 2: check every subarray using O(1) range-sum
        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                int sum = (i == 0) ? nums[j] : nums[j] - nums[i - 1];
                if (sum % k == 0) {
                    result++;
                }
            }
        }
        return result;
    }
};
```

**Note on the prefix loop:** it starts at `i = 1`, not `i = 0`, because `nums[0]` is already its own prefix sum — there is nothing before it to add.

### Complexity

| | |
|---|---|
| Time | **O(n²)** — two nested loops, O(1) work inside |
| Space | **O(1)** extra (the array is modified in place) |

### Result on LeetCode

**Time Limit Exceeded.**

### Why this step still matters (interview point)

The instructor makes an explicit interview argument here:

> Interviewers love gradual improvement. You state O(n³), then you improve to O(n²), then you go to the optimal. It creates a very good impression that the candidate can walk from the worst approach to the best approach on their own.

So even though O(n²) fails the judge, **saying it out loud in an interview is valuable**. It also directly motivates the final solution: we've squeezed the sum computation down to O(1), so the only remaining fat is the nested loop itself — which means we now need an idea that avoids checking every pair.

---

## 4. Approach 3 — Optimal, O(n) using Modulo + HashMap

> On-screen header: **`3. Optimal Approach :- O(n)`** — *"Modulo (%) बाबा की जय !!!"*

The instructor emphasises that he will **not** just hand you "use a map and store remainders." He first proves the underlying mathematics, then shows **why** a map naturally becomes necessary. This reasoning is the part you are expected to reproduce in an interview.

### 4.1 The mathematical lemma (with proof)

**Setup.** Picture an array split into labelled cells `a b c d e`.

- Let `S1` = sum of the segment from `a` up to `e` (the longer prefix).
- Let `S2` = sum of the segment from `a` up to `c` (the shorter prefix).
- The "leftover" middle portion is therefore `d + e`, whose sum equals **`S1 − S2`**.

**Given:** when `S1` is divided by `K`, the remainder is `x`; and when `S2` is divided by `K`, the remainder is **also `x`** (the same remainder).

**Claim:** then `S1 − S2` — i.e. the leftover subarray — is **guaranteed** divisible by `K`.

**Proof.** By definition of division with remainder we can write:

```
S1 = K·n  + x        (for some integer n)
S2 = K·n' + x        (for some integer n')
```

Subtract:

```
S1 − S2 = (K·n + x) − (K·n' + x)
        = K·n + x − K·n' − x
        = K(n − n')          ... the +x and −x cancel
        = K·N                ... where N = (n − n') is just some integer
```

Since `S1 − S2` can be written as `K` multiplied by an integer, it is **divisible by K**. ∎

The instructor gives an everyday sanity check: if you can write `a = 2 × 5`, then obviously `a` is divisible by 2. Same logic.

**The takeaway, in plain words:**

> If two prefix sums leave the **same remainder** when divided by K, then the subarray **between** them is divisible by K.

This is the entire engine of the optimal solution.

### 4.2 Why a HashMap is needed (the reasoning, not the recipe)

Now trace the consequence of the lemma:

1. We are going to walk left-to-right keeping a **running sum** (`sum`), adding one element at a time. → *That's why we need a `sum` variable.*
2. At each position we compute `sum % K` to get the remainder at that point.
3. The lemma says: if the remainder we just computed has been **seen earlier**, we've found a divisible subarray.
4. But to know "have I seen remainder `z` before?" — and, crucially, **how many times** — we must have **stored** the remainders we encountered.

> *That's why we use a map: to store each remainder and its count.*

The **count** matters, not just a yes/no flag: if a remainder has occurred 3 times before, the current position pairs up with **each** of those 3 earlier positions, producing 3 distinct divisible subarrays.

### 4.3 Why the map is initialised with `mp[0] = 1`

Before processing any element, the running sum is `0`. And `0 % K == 0`. So the remainder `0` has, in a sense, **already been seen once** — by the empty prefix.

This is not a hack; it is what makes subarrays that **start at index 0** get counted. If the prefix sum up to index `j` is itself divisible by K, it pairs with this initial empty prefix.

So we always begin with:

```cpp
mp[0] = 1;
```

---

## 5. Dry Run 1 — Positive Array

```
Index :  0   1   2   3   4   5   6
Value :  2   3   5   4   5   3   4        K = 7
```

Start: `sum = 0`, `result = 0`, map = `{0 : 1}` (remainder 0 seen once — the empty prefix).

| Step | Element | Running `sum` | `rem = sum % 7` | Seen before? | Action on `result` | Map after |
|---|---|---|---|---|---|---|
| init | — | 0 | 0 | — | — | `{0:1}` |
| i=0 | 2 | 2 | 2 | No | — | `{0:1, 2:1}` |
| i=1 | 3 | 5 | 5 | No | — | `{0:1, 2:1, 5:1}` |
| i=2 | 5 | 10 | 3 | No | — | `{0:1, 2:1, 5:1, 3:1}` |
| i=3 | 4 | 14 | 0 | **Yes, count 1** | `result += 1` → **1** | `{0:2, ...}` |
| i=4 | 5 | 19 | 5 | **Yes, count 1** | `result += 1` → **2** | `{5:2, ...}` |
| i=5 | 3 | 22 | 1 | No | — | `{1:1, ...}` |
| i=6 | 4 | 26 | 5 | **Yes, count 2** | `result += 2` → **4** | `{5:3, ...}` |

**Final `result` = 1 + 1 + 2 = 4.**

Final map state (matches the on-screen table):

| Remainder | Count evolution |
|---|---|
| 0 | 1 → 2 |
| 2 | 1 |
| 5 | 1 → 2 → 3 |
| 3 | 1 → 2 |
| 1 | 1 |

### Verifying each hit — *which* subarray was actually found

The instructor verifies every single addition, because this is where intuition is built.

**At i = 3 (`sum = 14`, `rem = 0`, count was 1):**
Remainder 0 was last seen at the *empty prefix* (the very start). So the leftover portion is the whole stretch from index 0 to index 3:
`2 + 3 + 5 + 4 = 14`, and `14 % 7 == 0` ✔

**At i = 4 (`sum = 19`, `rem = 5`, count was 1):**
Remainder 5 was previously seen at index 1 (where `sum` was 5). By the lemma, the middle part between them — indices 2 to 4 — must be divisible by 7:
`5 + 4 + 5 = 14`, and `14 % 7 == 0` ✔
This is the lemma in action: prefix-with-rem-5 minus prefix-with-rem-5 = divisible.

**At i = 6 (`sum = 26`, `rem = 5`, count was 2) — the key step:**
Remainder 5 had occurred **twice** before (at index 1 with sum 5, and at index 4 with sum 19). That means the current position forms a divisible subarray with **each** of them — hence `+2`, not `+1`. Both are real:

- Pairing with index 1 → subarray indices **2 to 6**: `5 + 4 = 9, +5 = 14, +3 = 17, +4 = 21`. `21 % 7 == 0` ✔
- Pairing with index 4 → subarray indices **5 to 6**: `3 + 4 = 7`. `7 % 7 == 0` ✔

This is exactly why the map stores a **count** and why we do `result += mp[rem]` rather than `result++`.

### After adding to result, always update the count

Note the ordering in every step: **first** add the existing count to `result`, **then** increment the count for the current remainder (`mp[rem]++`). The current prefix must be available for *future* positions to pair with, but it must not pair with itself.

---

## 6. The Critical Edge Case — Negative Remainders

This is the part the instructor calls out as the make-or-break detail, and he candidly admits: *"I couldn't catch this test case either — I had wrong submissions, and only learned it after looking at others' solutions. It's totally fine if you missed it. What matters is that you understand it now."*

### 6.1 Test case that exposes the bug

```
Index :  0   1   2   3   4   5   6   7
Value :  2  -6   3   1   2   8   2   1        K = 7
```

### 6.2 Dry run WITHOUT the fix (the buggy run)

In C++ (and Java), `%` on a negative number yields a **negative remainder**: `-4 % 7` is `-4`, not `3`.

| Step | Running `sum` | `rem = sum % 7` (raw) | Seen? | `result` | Map |
|---|---|---|---|---|---|
| init | 0 | 0 | — | 0 | `{0:1}` |
| i=0 (2) | 2 | 2 | No | 0 | `{0:1, 2:1}` |
| i=1 (−6) | −4 | **−4** | No | 0 | `{−4:1, ...}` |
| i=2 (3) | −1 | **−1** | No | 0 | `{−1:1, ...}` |
| i=3 (1) | 0 | 0 | Yes (1) | **1** | `{0:2, ...}` |
| i=4 (2) | 2 | 2 | Yes (1) | **2** | `{2:2, ...}` |
| i=5 (8) | 10 | 3 | **No** ← BUG | 2 | `{3:1, ...}` |

**Verifying the correct hits so far:**

- **i = 3:** remainder 0 previously seen at the empty prefix → subarray indices 0..3: `2 − 6 = −4, +3 = −1, +1 = 0`. Sum 0, and `0 % 7 == 0` ✔
- **i = 4:** remainder 2 previously seen at index 0 (sum 2) → subarray indices 1..4: `−6 + 3 = −3, +1 = −2, +2 = 0`. Sum 0, divisible ✔

**The miss at i = 5:** we computed `rem = 3` and concluded "never seen before." **That is wrong.** There *is* a valid subarray ending here:

Subarray indices **2..5**: `3 + 1 = 4, +2 = 6, +8 = 14`. And `14 % 7 == 0` ✔ — but we never counted it.

**Why was it missed?** The prefix at index 1 had raw remainder `−4`, which we stored as `−4`. But mathematically `−4` and `3` are the **same remainder class mod 7**. Because we stored the negative form, the map lookup for `3` failed to match it.

### 6.3 The algebra that fixes it

Take a sum of the form `7n − 4` (remainder `−4`). Add and subtract 7 — legal, since it changes nothing:

```
7n − 4
= 7n − 4 + 7 − 7
= (7n − 7) + (7 − 4)        ... group the two 7-multiples together
= 7(n − 1) + 3              ... factor 7 out of (7n − 7); and 7 − 4 = 3
```

So `7n − 4` can equally be written as `7 × (some integer) + 3`. The same number therefore has remainder **3**.

Observe where the `3` came from: it is `K + (−4)` = `7 + (−4)` = `3`. In other words, we simply **added K to the negative remainder**.

**General rule:**

```cpp
if (rem < 0) rem += K;
```

Since a raw remainder always satisfies `−K < rem < K`, a single `+K` is enough to bring any negative remainder into the range `[0, K−1]`. We must **never store a negative remainder** — we always normalise to the positive form so that all members of the same remainder class collide in the map correctly.

---

## 7. Dry Run 2 — Same Array, WITH the `rem += K` Fix

```
Index :  0   1   2   3   4   5   6   7
Value :  2  -6   3   1   2   8   2   1        K = 7
```

| Step | Element | `sum` | raw `rem` | fixed `rem` | Seen? (count) | `result` | Map after |
|---|---|---|---|---|---|---|---|
| init | — | 0 | 0 | 0 | — | 0 | `{0:1}` |
| i=0 | 2 | 2 | 2 | 2 | No | 0 | `{0:1, 2:1}` |
| i=1 | −6 | −4 | −4 | **−4+7 = 3** | No | 0 | `{3:1, ...}` |
| i=2 | 3 | −1 | −1 | **−1+7 = 6** | No | 0 | `{6:1, ...}` |
| i=3 | 1 | 0 | 0 | 0 | Yes (1) | **1** | `{0:2, ...}` |
| i=4 | 2 | 2 | 2 | 2 | Yes (1) | **2** | `{2:2, ...}` |
| i=5 | 8 | 10 | 3 | 3 | **Yes (1)** ← now caught! | **3** | `{3:2, ...}` |
| i=6 | 2 | 12 | 5 | 5 | No | 3 | `{5:1, ...}` |
| i=7 | 1 | 13 | 6 | 6 | Yes (1) | **4** | `{6:2, ...}` |

**Final `result` = 4.**

Final map state (matches the on-screen table):

| Remainder | Count evolution |
|---|---|
| 0 | 1 → 2 |
| 2 | 1 → 2 |
| 3 | 1 → 2 |
| 6 | 1 → 2 |
| 5 | 1 |

### Verification of the newly-caught hit at i = 5

Remainder 3 was first recorded at index 1 (raw sum `−4`, normalised to 3). The current position has remainder 3 as well. By the lemma, the portion strictly between them — indices **2 to 5** — must be divisible by 7:

`3 + 1 = 4, +2 = 6, +8 = 14` → `14 % 7 == 0` ✔

Previously missed, now correctly counted. That single `rem += K` line recovered a real answer.

### Verification of the hit at i = 7

Remainder 6 was first seen at index 2 (raw sum `−1` → normalised 6). Pairing gives indices **3 to 7**: `1 + 2 + 8 + 2 + 1 = 14` → divisible by 7 ✔

### Arithmetic checks used in the lecture
- `12 % 7` → `7 × 1 = 7`, `12 − 7 = 5` → remainder **5**
- `13 % 7` → `13 − 7 = 6` → remainder **6**
- `26 % 7` → `7 × 3 = 21`, `26 − 21 = 5` → remainder **5**

---

## 8. Final Optimal Code (Accepted)

```cpp
class Solution {
public:
    int subarraysDivByK(vector<int>& nums, int k) {
        int n = nums.size();

        unordered_map<int, int> mp;   // remainder -> how many times seen
        int sum = 0;
        mp[0] = 1;                    // empty prefix: sum 0, remainder 0

        int result = 0;

        for (int i = 0; i < n; i++) {
            sum += nums[i];           // running prefix sum

            int rem = sum % k;        // raw remainder (can be negative)

            if (rem < 0) {            // normalise into [0, k-1]
                rem += k;
            }

            if (mp.find(rem) != mp.end()) {   // have we seen this remainder before?
                result += mp[rem];            // pair with EVERY earlier occurrence
            }

            mp[rem]++;                // record the current prefix for future pairings
        }

        return result;
    }
};
```

### Line-by-line reasoning

| Line | Why it's there |
|---|---|
| `unordered_map<int,int> mp;` | Stores remainder → frequency. `unordered_map` (hash map) gives **O(1)** average lookup/insert, which is what keeps the whole solution linear. |
| `mp[0] = 1;` | Before any element, `sum = 0` and `0 % k == 0`. Seeding this lets subarrays starting at index 0 be counted. |
| `sum += nums[i];` | Maintains the prefix sum incrementally — no recomputation, O(1) per step. |
| `int rem = sum % k;` | The remainder is the "signature" of this prefix. |
| `if (rem < 0) rem += k;` | **The critical edge case.** Without it, `−4` and `3` (mod 7) are treated as different keys and valid subarrays are silently missed. |
| `if (mp.find(rem) != mp.end())` | Checks whether this remainder class has appeared before. Using `find` avoids accidentally inserting a default `0` entry via `mp[rem]` during the check. |
| `result += mp[rem];` | **Add the count, not 1.** If the remainder occurred `c` times before, the current prefix forms `c` distinct divisible subarrays. |
| `mp[rem]++;` | Register the current prefix *after* counting, so it can pair with future prefixes but never with itself. |
| `return result;` | Total number of qualifying subarrays. |

### Execution flow in one sentence

Walk once through the array maintaining a running prefix sum; at each index compute the normalised remainder mod K; add to the answer the number of earlier prefixes sharing that remainder (each such pairing is a divisible subarray by the lemma); then record the current remainder.

### Complexity

| | |
|---|---|
| **Time** | **O(n)** — a single pass with O(1) hash-map operations |
| **Space** | **O(K)** in the worst case (at most K distinct remainder keys `0 … K−1`); commonly stated as O(n) upper bound |

### Submission Result (on screen)

- Status: **Accepted**
- Runtime: **75 ms** (faster than 55.61% of C++ submissions)
- Memory: **31.6 MB**

---

## 9. Summary of All Three Approaches

| Approach | Core Idea | Time | Space | LeetCode Verdict |
|---|---|---|---|---|
| 1. Brute force | Generate every subarray, sum it from scratch | O(n³) | O(1) | TLE |
| 2. Prefix sum + nested loops | Precompute cumulative sums so each range-sum is O(1) | O(n²) | O(1) | TLE |
| 3. Prefix remainder + HashMap | Same remainder ⇒ divisible middle segment; count pairs in one pass | **O(n)** | O(K) | **Accepted** |

---

## 10. Interview-Relevant Takeaways & Common Mistakes

**Things to say / do in an interview**
- Present the progression **O(n³) → O(n²) → O(n)**. The instructor stresses that interviewers are impressed by a candidate who can move from the worst approach to the best on their own. Don't jump straight to the optimal without acknowledging the baseline.
- Be ready to **prove** the lemma (`S1 = Kn + x`, `S2 = Kn' + x` ⇒ `S1 − S2 = K(n − n')`), not just assert it. The instructor deliberately refuses to state it without proof: *"don't trust a statement just because someone said it."*
- Be able to explain **why a map appeared in your head** — i.e. that you need to remember previously-seen remainders *and their counts* — rather than presenting it as a memorised template.

**Bugs and traps to watch for**
1. **Negative remainders.** In C++/Java, `%` on a negative value returns a negative remainder. Always do `if (rem < 0) rem += K;`. Forgetting this silently undercounts and is the single most common failure on this problem.
2. **`result += mp[rem]`, not `result++`.** Adding 1 instead of the stored count loses all but one pairing per position.
3. **`mp[0] = 1` initialisation.** Omitting it loses every subarray that starts at index 0.
4. **Order of operations:** add to `result` *before* incrementing `mp[rem]`, otherwise a prefix pairs with itself and you overcount by 1 each step.
5. **`nums[i-1]` when `i == 0`** in the O(n²) approach — out-of-bounds. Guard with the `i == 0` ternary/if.
6. **Remember `0` is divisible by K**, and **negative sums can be divisible too** (`−5 % 5 == 0`).

**Study habit the instructor emphasises**
- **Always do a dry run.** He runs the full dry run twice — once to expose the negative-remainder bug, once with the fix — because *"dry run karne se hi best samajh me aata hai."*
- Understand the *reason* behind each line. Watching a solution video or copying code doesn't build the skill; the reasoning is what transfers to the next problem.
