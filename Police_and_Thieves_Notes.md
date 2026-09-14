# Police and Thieves — Complete Study Notes

**Source:** GeeksforGeeks Practice — "Police and Thieves"
**Difficulty:** Medium · **Accuracy:** 51.48% · **Submissions:** 7264 · **Points:** 4
**Asked by:** Microsoft and other large product companies.

The instructor positions this problem right after *Stock Buy and Sell* and *Jump Game*. He notes that this one is **slightly easier** than those, but it is still a very good interview question because the optimal solution needs a clean greedy insight plus a two-pointer implementation.

---

## 1. Problem Statement

You are given a **character array** `arr[]` of size `n`. Every element is either:

- `'P'` → a policeman
- `'T'` → a thief

Find the **maximum number of thieves that can be caught** by the police.

### Conditions

1. **Each policeman can catch only one thief.** (One-to-one matching — a policeman cannot catch two thieves, and a thief cannot be caught by two policemen.)
2. **A policeman cannot catch a thief who is more than `K` units away from him.**
   `K` is given in the input.

### What "K units away" means

Distance is measured on **indices**, and it works in **both directions** — ahead *and* behind.

If a policeman stands at index `i`, he can catch any thief sitting in the index window:

```
[ i - k , i + k ]
```

So direction does not matter. The instructor stresses: *"आगे-पीछे कैसे भी आपको बस चोर को पकड़ना है and maximize करना है"* — catch in front or behind, it doesn't matter; you just have to **maximize** the count.

---

## 2. Given Examples (from the GFG page)

### Example 1

```text
Input:
N = 5, K = 1
arr[] = {P, T, T, P, T}
Output: 2
```

**Explanation (as walked through in the lecture):**

| Index | 0 | 1 | 2 | 3 | 4 |
|-------|---|---|---|---|---|
| char  | P | T | T | P | T |

- Police at index `0`: his range is `[-1, 1]`. Index `-1` doesn't exist, so only index `1` is usable. Index `1` is a thief → he catches that thief.
- Police at index `3`: his range is `[2, 4]`. Both index `2` and index `4` are thieves, so **two options are available**; he catches any **one** of them (only one allowed).
- Total thieves caught = **2**.

### Example 2

```text
Input:
N = 6, K = 2
arr[] = {T, T, P, P, T, P}
Output: 3
```

| Index | 0 | 1 | 2 | 3 | 4 | 5 |
|-------|---|---|---|---|---|---|
| char  | T | T | P | P | T | P |

- Police at index `2` catches the thief at index `0` (distance 2 ≤ 2).
- Police at index `3` catches the thief at index `1` (distance 2 ≤ 2).
- Police at index `5` catches the thief at index `4` (distance 1 ≤ 2).
- Total = **3**. Every police and every thief gets paired here.

---

## 3. Constraints and Expected Complexity (shown on screen)

```text
Expected Time Complexity  : O(N)
Expected Auxiliary Space  : O(N)

Constraints:
1 <= N <= 10^5
1 <= K <= 100
arr[i] = 'P' or 'T'
```

Two things to notice immediately, because they drive the whole discussion:

- `N` can be up to **10^5**.
- `K` is capped at a very small **100**. This small cap is exactly why the brute-force solution still passes.
- The **expected** solution is `O(N)` time and `O(N)` auxiliary space — note that the problem *allows* extra space, which is a hint that building helper arrays is acceptable.

---

## 4. The Core Insight (the "why" of the greedy)

This is the most important part of the lecture — everything else is implementation.

**Question posed by the instructor:** when a policeman has several thieves inside his range, *which one should he catch?*

**Answer (Shiv's reasoning, confirmed by the instructor):**
He should catch the thief who is **farthest back in his reach** — i.e. the thief with the **smallest index** inside `[i-k, i+k]`.

### Why? (the intuition)

Think about who else could possibly catch that far-behind thief.

- Policemen are processed **left to right**. The policemen that come *after* the current one sit at **larger indices**, so they are even **farther away** from a thief who is already at the left edge of the current policeman's reach.
- Therefore, if the **current** policeman does not catch that leftmost reachable thief, **nobody ever will**. That thief is permanently lost.
- Meanwhile, a thief who is closer/ahead still has a chance of being caught by some later policeman.

The instructor's exact framing:

> *"अगर यह पुलिस इस चोर को पकड़ लेता, तो फिर यह पुलिस क्या करता? वेस्ट हो जाता, हम लोगों को maximize करना है।"*
> — If this policeman grabs the *nearer/ahead* thief, then the *next* policeman ends up with nothing to catch and gets **wasted**. And our goal is to *maximize*, so we cannot afford wasted policemen.

So the greedy rule is:

> **Pair the earliest available policeman with the earliest available thief, whenever they are within distance K.**

This is a classic **greedy matching / two-pointer** pattern: matching the smallest available item on the left with the smallest available item on the right never hurts, because keeping a "small" element around only makes it harder to match later.

---

## 5. Approach 1 — Brute Force (Shiv's approach): O(N × K)

### Algorithm

1. Iterate `i` over the array from left to right.
2. The moment you find `arr[i] == 'P'` (a policeman), **stop and enter an inner loop**.
3. Inner loop scans the window from `i - k` to `i + k` (clamped to valid array bounds `0 .. n-1`), **starting from the left end (`i - k`)**.
4. The **first** `'T'` you encounter in that left-to-right scan is the **leftmost reachable thief** — exactly the one the greedy insight says to catch.
5. Increment the answer counter and **`break`** out of the inner loop (one policeman = one thief).
6. **Mark that thief as caught** — overwrite `arr[that_index]` with some other character, e.g. `'S'` (or anything that is not `'T'`).
7. Continue the outer loop from where you left off; when you meet the next `'P'`, repeat the same window scan.

### Why the marking step is compulsory (important warning)

If you do **not** overwrite the caught thief's `'T'`, then a **later policeman whose window also covers that same index could catch the same thief again**. That would double-count one thief and produce a wrong (inflated) answer.

The instructor explains it directly: change that `T` into `S` (or literally anything else) so that *"यह T हट जाए"* — the `T` disappears from the array — and the same thief can never be picked twice.

### Why scanning from `i-k` (and not from `i+k`) matters

The direction of the inner scan is not cosmetic — it **is** the greedy. Scanning left-to-right from `i-k` guarantees the first hit is the leftmost (most "at-risk") thief. If you scanned from `i+k` backwards, you would greedily grab the thief that later policemen could still have handled, and the answer would be suboptimal.

### Complexity

- **Time:** For every policeman you scan a window of size `2K + 1`. Worst case that is `O(N × K)`.
  Plugging in the constraints: `10^5 × 10^2 = 10^7` operations.
- **Space:** `O(1)` — no extra data structures, only in-place marking.

### Does it pass?

**Yes.** The instructor explicitly reasons about this: `10^7` is well under the `~10^8` operations-per-second rule of thumb, so *"कोई दिक्कत नहीं है"* — no problem, it will run in time.

**But** the question's *expected* complexity is `O(N)` time with `O(N)` space, and in an interview you would be asked to do better than `N*K`. That is what motivates Approach 2.

> The instructor deliberately does **not** write this brute-force code on screen — he asks the viewer to implement it themselves as practice: *"मैं नहीं बताऊँगा, आप खुद से लिखने की कोशिश कीजिए."*

---

## 6. Approach 2 — Optimal Two-Pointer: O(N) time, O(N) space

### Step 1: Separate the indices into two arrays

Instead of scanning windows, **store the positions** of policemen and thieves in two separate arrays.

Take the whiteboard example `T T P P T P`:

| Index | 0 | 1 | 2 | 3 | 4 | 5 |
|-------|---|---|---|---|---|---|
| char  | T | T | P | P | T | P |

```text
police[] = { 2, 3, 5 }
thief[]  = { 0, 1, 4 }
```

**Critical property:** both arrays are **automatically sorted in increasing order**, because we build them by pushing indices while sweeping `i` from `0` to `n-1`. The instructor calls this out explicitly:

> *"यह array दोनों by default sorted है, क्योंकि हम लोग index push back कर रहे हैं तो sorted तो है"*

No extra sorting step is ever needed — this is what makes the two-pointer legal.

**Space used:** in the worst case the two arrays together hold all `N` indices (e.g. `N/2 + N/2`), so the space is `O(N)` — which is exactly what the problem allows.

### Step 2: Run two pointers

Keep pointer `i` on `police[]` and pointer `j` on `thief[]`, both starting at `0`, and a counter `ans = 0`.

At each step compare the current policeman's index with the current thief's index using **absolute difference** (because catching works in both directions):

#### Case A — `abs(police[i] - thief[j]) <= k` → **Catch!**

They are within reach. Match them.
`ans++`, `i++`, `j++` — **both** pointers move, because this policeman is now used up *and* this thief is now caught.

#### Case B — `police[i] > thief[j]` (and out of range) → **`j++` (advance the thief)**

The thief is sitting **behind** the policeman and is still too far away.

**Reasoning (the key argument):** the current policeman cannot reach him. Can any *later* policeman reach him? **No** — every later policeman has an even **larger** index, so he is even **farther** from this thief. Therefore **this thief can never be caught by anybody**. Discard him and move on.

The instructor repeats this at the end of the video to make sure it lands:

> Suppose police is at `3` and thief is at `0`, and `3` can't reach `0`. *"इसके आगे जो पुलिस है वो कभी भी इस चोर को पकड़ पाएगा? जब यही नहीं पकड़ पा रहा तो इसके आगे कहाँ से पकड़ेगा?"* — If even this policeman can't reach him, a policeman further right certainly can't. **This thief will never be caught.** So `j++`.

#### Case C — `police[i] < thief[j]` (and out of range) → **`i++` (advance the police)**

The thief is **ahead** of the policeman and too far away.

**Reasoning:** this policeman cannot reach the current thief. Can he reach any *later* thief? **No** — later thieves have even **larger** indices, so they are even farther ahead. Therefore **this policeman is useless** and can be discarded. Move to the next policeman.

> *"अगर इसको ही नहीं पकड़ पा रहा, तो उसके आगे वाले चोर को तो पकड़ ही नहीं सकता... पुलिस को बढ़ा दूँगा"*

#### The rule as written on the whiteboard

```text
P_ind  <  T_ind   ->   P++      (police index smaller, out of range → move police)
P_ind  >  T_ind   ->   T++      (police index larger,  out of range → move thief)
in range          ->   both++ , ans++
```

Equivalent phrasing used in the lecture (in terms of the window): if the thief's index is greater than `i + k`, move the **police**; if the thief's index is smaller than `i - k`, move the **thief**.

#### Loop termination

Stop as soon as **either** pointer runs off its array — if you run out of policemen there is nobody left to catch, and if you run out of thieves there is nobody left to be caught.

---

## 7. The Code (C++, as written on screen)

```cpp
class Solution{
    public:
    int catchThieves(char arr[], int n, int k) 
    {
        // Step 1: collect indices
        vector<int> police, thief;
        for(int i=0; i<n; i++)
        {
            if(arr[i]=='P')
                police.push_back(i);
            else
                thief.push_back(i);
        }

        // Step 2: two pointers
        int i=0, j=0;
        int ans=0;
        while(i<police.size() && j<thief.size())
        {
            if(abs(police[i]-thief[j])<=k)   // in range → catch
            {
                ans++; i++; j++;
            }
            else if(police[i]>thief[j])      // thief too far behind → he is lost forever
            {
                j++;
            }
            else                             // police too far behind the thief → this police is useless
            {
                i++;
            }
        }
        return ans;
    }
};
```

### Line-by-line explanation

| Line / Block | What it does and why |
|---|---|
| `vector<int> police, thief;` | Two containers to hold **indices** (not characters). |
| `for(int i=0; i<n; i++)` | Single left-to-right sweep over the input array. |
| `if(arr[i]=='P') police.push_back(i); else thief.push_back(i);` | Since only `'P'` and `'T'` exist, `else` safely means thief. Pushing in increasing `i` keeps both vectors **sorted**. |
| `int i=0, j=0;` | `i` walks `police[]`, `j` walks `thief[]`. |
| `int ans=0;` | Count of successful catches. |
| `while(i<police.size() && j<thief.size())` | Stop as soon as either side is exhausted. |
| `abs(police[i]-thief[j])<=k` | **Absolute** difference because a policeman catches in **both** directions. `<= k` (not `< k`) — exactly `k` units away is still catchable. |
| `ans++; i++; j++;` | One catch consumes **one** policeman **and** one thief — this is what enforces "each policeman catches only one thief". |
| `else if(police[i]>thief[j]) j++;` | Thief is behind and unreachable → unreachable by all future (further-right) policemen too → drop the thief. |
| `else i++;` | Thief is ahead and unreachable → all future (further-right) thieves are worse → drop the policeman. |
| `return ans;` | Maximum number of thieves caught. |

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | **O(N)** | One pass to build the arrays + one two-pointer pass in which **every iteration advances at least one pointer**, so at most `|police| + |thief| = N` iterations. |
| **Space** | **O(N)** | The two index vectors together hold at most `N` entries (`N/2 + N/2` in a balanced case). |

This exactly matches the problem's expected `O(N)` / `O(N)`.

---

## 8. Dry Runs

### Dry Run 1 — `T T P P T P`, K = 2

```text
police[] = { 2, 3, 5 }
thief[]  = { 0, 1, 4 }
```

| Step | i | j | police[i] | thief[j] | abs diff | ≤ K? | Action | ans |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 2 | 0 | 2 | Yes | catch, i++, j++ | 1 |
| 2 | 1 | 1 | 3 | 1 | 2 | Yes | catch, i++, j++ | 2 |
| 3 | 2 | 2 | 5 | 4 | 1 | Yes | catch, i++, j++ | 3 |
| 4 | 3 | 3 | — | — | — | — | `j` (and `i`) exhausted → exit | 3 |

**Answer = 3** ✔ (matches GFG Example 2, and the whiteboard arrows `2 → 0`, `3 → 1`, `5 → 4`.)

**Insight highlighted here:** at every step the policeman is matched with the **earliest** remaining thief, which is precisely the greedy rule.

---

### Dry Run 2 — `P P T T` (the instructor's modified case)

```text
police[] = { 0, 1 }
thief[]  = { 2, 3 }
```

**With K = 2:**

| Step | i | j | police[i] | thief[j] | abs diff | ≤ 2? | Action | ans |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 0 | 2 | 2 | Yes | catch | 1 |
| 2 | 1 | 1 | 1 | 3 | 2 | Yes | catch | 2 |

**Answer = 2.** The instructor's point: *"यह 0, सबसे best option क्या है? यह इसको पकड़े"* — the best option for police `0` is the earliest thief `2`; then both move forward and police `1` takes thief `3`.

**Now the interesting variation — K = 1:**

| Step | i | j | police[i] | thief[j] | abs diff | ≤ 1? | Comparison | Action | ans |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 0 | 2 | 2 | **No** | `police(0) < thief(2)` | **i++** (police is useless) | 0 |
| 2 | 1 | 0 | 1 | 2 | 1 | Yes | — | catch, i++, j++ | 1 |
| 3 | 2 | 1 | — | — | — | — | `i` exhausted | exit | 1 |

**Answer = 1.** This is the case that demonstrates **Case C**: police at index `0` can never reach thief `2` (and certainly not thief `3`, which is even farther), so we throw the policeman away, not the thief.

---

### Dry Run 3 — `P T T P P P T T P`, K = 2 (the longer whiteboard case)

| Index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|-------|---|---|---|---|---|---|---|---|---|
| char  | P | T | T | P | P | P | T | T | P |

```text
police[] = { 0, 3, 4, 5, 8 }
thief[]  = { 1, 2, 6, 7 }
```

| Step | i | j | police[i] | thief[j] | abs diff | ≤ 2? | Action | ans |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 0 | 1 | 1 | Yes | catch | 1 |
| 2 | 1 | 1 | 3 | 2 | 1 | Yes | catch | 2 |
| 3 | 2 | 2 | 4 | 6 | 2 | Yes | catch | 3 |
| 4 | 3 | 3 | 5 | 7 | 2 | Yes | catch | 4 |
| 5 | 4 | 4 | 8 | — | — | — | `j` exhausted → exit | 4 |

**Answer = 4** ✔ (matches the whiteboard matching arrows `0 → 1`, `3 → 2`, `4 → 6`, `5 → 7`.)

**Note:** the policeman at index `8` is simply left unused — there are no thieves left. That is fine; the answer is bounded by `min(#police, #thieves)` anyway.

**Hypothetical variation discussed:** the instructor asks — if the thief's index had been something like `-3` while police is at `0` (impossible in reality, purely for illustration), which pointer would we advance? **The thief's pointer**, because the thief is *too far behind* and no later (further-right) policeman could ever reach him. This is **Case B**.

---

## 9. Edge Cases, Pitfalls and Warnings

1. **Window must be clamped to array bounds (brute force).** `i - k` can go negative and `i + k` can exceed `n - 1`. In Example 1, police at index `0` with `K = 1` has no valid index `-1`.
2. **Never forget to mark a caught thief in the brute-force version.** Without overwriting `'T'` → `'S'`, one thief can be counted by two policemen.
3. **Use `<= k`, not `< k`.** A thief *exactly* `k` units away **is** catchable. In Example 2, `|2 - 0| = 2` with `K = 2` is a valid catch.
4. **Use absolute difference.** Catching works forward *and* backward; a one-sided comparison is wrong.
5. **Both pointers move only on a successful catch.** On a failed comparison, exactly **one** pointer moves — this is what guarantees `O(N)` and also prevents skipping a valid pairing.
6. **Do not sort the index arrays.** They are already sorted by construction; sorting would waste `O(N log N)`.
7. **Signed/unsigned comparison:** `i < police.size()` compares an `int` with `size_t`, which produces a compiler warning. Use `(int)police.size()` if you want warning-free code.
8. **Direction of the inner scan in brute force is part of the algorithm** — scan from `i-k` upward, not from `i+k` downward.
9. **Live-demo note:** the GFG site threw a `404 — page missing/unavailable` error when the instructor tried to run the code on the platform. He confirms the **logic and code are correct** and advises running it in your own compiler; the code link is given in the video description. This was a website issue, not a bug.

---

## 10. Comparison of the Two Approaches

| | Approach 1 (Brute force) | Approach 2 (Two pointers) |
|---|---|---|
| **Idea** | For each `P`, scan `[i-k, i+k]` left→right, catch the first `T`, mark it | Store indices of `P` and `T` in two sorted arrays, match greedily with two pointers |
| **Time** | `O(N × K)` ≈ `10^7` for the given constraints | `O(N)` |
| **Space** | `O(1)` | `O(N)` |
| **Passes the constraints?** | Yes (`10^7` < `10^8`) | Yes |
| **Meets "expected" complexity?** | No (expected is `O(N)`) | Yes (`O(N)` time, `O(N)` aux space) |
| **Extra care needed** | Must mark caught thieves; must clamp bounds | Must pick the correct pointer to advance on a miss |

---

## 11. Quick Revision Summary

- **Pattern:** Greedy + Two Pointers (a matching problem between two sorted sequences).
- **Greedy claim:** always pair the **earliest unused policeman** with the **earliest uncaught thief** if they are within `K`.
- **Why the greedy is safe:** the leftmost remaining thief is the one *least* likely to be reachable by anyone later, so if the current (leftmost) policeman can take him, taking him never loses anything. Saving a policeman for a nearer thief only wastes a policeman.
- **Three-case decision:**
  - `|P - T| <= K` → catch: `ans++, i++, j++`
  - `P > T` → this thief is behind and unreachable by *everyone* remaining → `j++`
  - `P < T` → this policeman is behind and can reach *nobody* remaining → `i++`
- **Answer is bounded by** `min(number of policemen, number of thieves)`.
- **Final complexity:** `O(N)` time, `O(N)` space.
