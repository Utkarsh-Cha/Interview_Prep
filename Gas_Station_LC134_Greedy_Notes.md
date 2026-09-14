# Greedy — Video 12 : Gas Station (LeetCode 134)

**Problem:** LeetCode 134 — *Gas Station*
**Difficulty:** Medium
**Also known as:** *"Circular Tour"* (this is the name used on GFG)
**Asked by:** Google, Uber, Microsoft, Amazon, Zoho, FactSet, Morgan Stanley

**Instructor's framing:** This is a *rare*, weird-feeling question. Just reading the statement doesn't make it click — it only makes sense once you follow the "story" behind it. Questions of this flavour show up occasionally, mostly from Google. The instructor says plainly: *"I hate this question, but I also love this question"* — hate it because the intuition is non-obvious, love it because it teaches an enormous amount (brute force construction, greedy intuition, and a mathematical guarantee argument).

---

## 1. "आज का ज्ञान" — The Life Lesson That IS the Algorithm

Before touching the problem, the instructor gives a piece of life advice. It looks like a random tangent, but it turns out to be the **entire greedy insight** in disguise.

> **"अगर तुम in total जितना कमा रहे हो, उससे ज्यादा का खर्च करने का plan करते हो तो कभी अपने dream goal तक नहीं पहुँच पाओगे।"**
>
> *(If, in total, you plan to spend more than what you earn in total, you will never reach your dream goal.)*

**Explanation of the idea:**

- Your income may come from **many sources** — tuitions you teach, money parents send, an internship stipend, etc. Add them all up — that's your **total earning**.
- Now add up **all** your planned/future expenses — that's your **total expenditure**.
- If you earn ₹5 in total but plan to spend ₹10 in total, you can rearrange your spending however you like, budget however cleverly you like — you will **never** reach the goal. The shortfall is structural, not a matter of ordering.

On screen this was annotated simply as: `5 < 10`.

**Why it matters here:** In the Gas Station problem, "earning" = total gas available across all stations, "expense" = total cost to travel between all stations. Exactly the same logic decides whether an answer exists at all. Keep this in mind — we come back to it in Section 5.

*(The instructor also thanked Shrijal Acharya for a logo gift sent on LinkedIn — not part of the content.)*

---

## 2. Understanding the Problem Statement

### 2.1 The Setup

You are given `n` gas stations arranged **in a circle**, and two arrays:

```text
Index :    0    1    2    3    4          n = 5
gas   = [  1,   2,   3,   4,   5  ]
cost  = [  3,   4,   5,   1,   2  ]

Output : 3
```

- **`gas[i]`** = number of units of gas **present at / earned from** station `i`. The moment you arrive at (or start at) station `i`, you **gain** `gas[i]` units.
  - Station 0 has 1 unit, station 1 has 2 units, station 2 has 3 units, and so on.
- **`cost[i]`** = number of units of gas **charged / consumed** to travel **from station `i` to station `i+1`**.

### 2.2 The Movement Rule (stated precisely)

> If you are standing at the **i-th** station and you want to move to the **(i+1)-th** station, you will be charged **`cost[i]`** units of gas — the cost belongs to the station you are *leaving*, not the one you are going to.

So one "hop" is always two things happening:

```text
currentGas = currentGas - cost[i]      // pay to leave station i
currentGas = currentGas + gas[i+1]     // collect gas on arriving at station i+1
```

Because the stations are circular, after station `n-1` you go back to station `0`.

### 2.3 Worked Micro-Examples (from the lecture)

**Example A — a failing move (standing at index 2):**

- You start at index 2 → you have `gas[2] = 3` units.
- You want to go to station 3. Leaving station 2 costs `cost[2] = 5`.
- You have 3, you need 5 → **3 < 5 → you cannot move.** You are stuck.

**Example B — a successful move (standing at index 3):**

- You are at index 3 → you have `gas[3] = 4` units.
- To leave station 3, cost is `cost[3] = 1`. Since `4 ≥ 1`, you can move.
- `4 - 1 = 3` units left, and you arrive at station 4.
- Station 4 gives you `gas[4] = 5` → `3 + 5 = 8`. Now you hold **8** units.

**Example C — continuing from station 4 back to station 0 (wrap-around):**

- You hold 8. To leave station 4, cost is `cost[4] = 2` → `8 - 2 = 6`.
- You wrap around to station 0 and collect `gas[0] = 1` → `6 + 1 = 7`.

### 2.4 What the Question Actually Asks

> Return the **index of the starting station** such that, if you begin your journey there with an empty tank, you can travel around the **entire circle once** and come back to that same starting station.

Key points:

- You may start at **any** of the 5 stations — index 0, 1, 2, 3, or 4.
- **The full cycle must complete.** E.g. if you start at index 1: `1 → 2 → 3 → 4 → 0 → back to 1`. All of it.
- You must **never run out of gas mid-way**. If at any station your remaining gas is less than the cost to leave that station, you are stuck ("phas jaoge") and that start is invalid.
- If no such station exists, return `-1`.
- (LeetCode guarantees the answer is unique when it exists.)

---

## 3. Full Dry Run — Why the Answer Is Index 3

### 3.1 Why indices 0, 1 and 2 fail immediately

The check is the simplest possible: *can you even leave the station you started from?*

| Start index `i` | Gas you hold on starting (`gas[i]`) | Cost to leave (`cost[i]`) | Verdict |
|---|---|---|---|
| 0 | 1 | 3 | `1 < 3` → stuck at the very first hop ❌ |
| 1 | 2 | 4 | `2 < 4` → stuck ❌ |
| 2 | 3 | 5 | `3 < 5` → stuck ❌ |
| 3 | 4 | 1 | `4 ≥ 1` → can move ✔ (continue checking) |

The instructor phrases index 1 as: *"you got 2 gas at station 1, nice — but leaving costs 4. Your dream is broken here, so index 1 is definitely not the answer."*

### 3.2 Complete circular trace starting from index 3

Starting at index 3 with a fresh tank, we collect `gas[3] = 4`:

```text
Start at index 3           → tank = 4

3 → 4 :  pay cost[3] = 1   → 4 - 1 = 3
         collect gas[4]=5  → 3 + 5 = 8      (now at station 4)

4 → 0 :  pay cost[4] = 2   → 8 - 2 = 6
         collect gas[0]=1  → 6 + 1 = 7      (now at station 0)

0 → 1 :  pay cost[0] = 3   → 7 - 3 = 4
         collect gas[1]=2  → 4 + 2 = 6      (now at station 1)

1 → 2 :  pay cost[1] = 4   → 6 - 4 = 2
         collect gas[2]=3  → 2 + 3 = 5      (now at station 2)

2 → 3 :  pay cost[2] = 5   → 5 - 5 = 0
         BACK AT STATION 3 — full circle complete ✔
```

This is exactly the chain of numbers written on screen:

```text
4
4 - 1 = 3
3 + 5 = 8
8 - 2 = 6
6 + 1 = 7
7 - 3 = 4
4 + 2 = 6
6 - 4 = 2
2 + 3 = 5
5 - 5 = 0     →  Answer = 3  (circled on screen)
```

**Important detail the instructor highlights:** when you come back to station 3, you of course collect `gas[3] = 4` again — but that is irrelevant. The answer we report is **3**, the index we *started from*, not any gas value.

Notice also the tank hit exactly `0` on the last hop and that is still **valid** — the condition for being stuck is `currentGas < cost[i]`, i.e. strictly less. `5 - 5 = 0` means you *just barely* made it, which is fine.

---

## 4. Approach 1 — Brute Force (O(n²))

### 4.1 Why start with brute force at all

The instructor makes a strong interview point here:

- When a question doesn't click, **always go to brute force first**. Don't freeze.
- In interviews (especially Google), **they expect you to start from brute force.** Then they drop hints and push you toward the optimal solution. Writing a working brute force is itself a big achievement — 34/37 test cases passed here.
- Many YouTube solutions skip brute force because it makes the video longer; the instructor deliberately includes it because *you must be able to write it*. Writing a brute force is **not automatic** — it requires you to genuinely understand the question.
- The whole method here is: **tell the story in plain words, then translate the story into code, line by line.** That's literally all brute force is.

### 4.2 The story (algorithm in words)

1. The question asks for an index, and I don't know which one it is → **try every index**.
2. Sit at index `i`. Ask: *"starting here, can I go all the way around the circle and come back to `i`?"*
3. If yes → `i` is the answer, return it.
4. If no → move on and try `i + 1`.
5. If **no index** works → return `-1`.

To walk the circle we use a second pointer `j`:

- `j` starts at `i + 1`.
- We keep moving `j` forward **while `j != i`**.
- The moment `j` comes back around and equals `i`, we have covered every station → we won → the answer is `i`.
- Circular movement is done with the classic school trick: **`j = (j + 1) % n`**. Never write plain `j + 1` — you must wrap. If `j` is the last index, `(j+1) % n` sends you to `0`, which is exactly what you want.

### 4.3 Step-by-step derivation on the example (as done on the whiteboard)

```text
i = 0 : currGas = 1, cost = 3   →  1 < 3  → reject
i = 1 : currGas = 2, req  = 4   →  2 < 4  → reject
i = 2 : currGas = 3, cost = 5   →  3 < 5  → reject
i = 3 : currGas = 4, cost = 1   →  OK, start the j-loop
```

For `i = 3`:

- `j = (i + 1) % n = 4`.
- Before entering the loop we already perform the first hop:
  `currGas = gas[3] - cost[3] + gas[4] = 4 - 1 + 5 = 8`. So standing at station 4 we hold **8**.
- Now the `while (j != i)` loop runs. At each iteration the **first thing** we check is the break condition:
  `if (currGas < cost[j]) break;` — i.e. *"can I even leave this station?"* Here `currGas = 8`, `cost[4] = 2`, `8 ≥ 2`, so we do not break.
- Update: `currGas = currGas - cost[j] + gas[next j]` = `8 - 2 + 1 = 7` after moving to index 0.
- The loop continues in the same fashion until either `j` returns to `i` (success) or a break happens (failure).

The on-screen loop skeleton was:

```text
j = (i + 1) % n
while (j != i) {
    if (currGas < cost[j])
        break;
    currGas = 8 - cost[i] + 1 = 8 - 2 + 1 = 7
}
if (j == i)
    return i;

return -1;
```

### 4.4 Handwritten pseudo-code (exactly as developed on screen)

```cpp
for (int i = 0; i < n; i++) {
    if (gas[i] < cost[i])
        continue;

    int j = (i + 1) % n;
    int costForMovingFromThisStation = cost[i];
    int gasEarnInNextStationj = gas[j];

    int currGas = gas[i] - costForMovingFromThisStation + gasEarnInNextStationj;

    while (j != i) {
        if (currGas < cost[j])
            break;

        int costForMovingFromThisj = cost[j];
        j = (j + 1) % n;

        int gasEarnInNextStationj = gas[j];

        currGas = currGas - costForMovingFromThisj + gasEarnInNextStationj;
    }

    if (j == i)
        return i;
}
return -1;
```

### 4.5 Final Brute Force C++ code (as typed into LeetCode)

```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int n = gas.size();

        for (int i = 0; i < n; i++) {
            if (gas[i] < cost[i])
                continue;

            int j = (i + 1) % n;                       // gola ghoome ke liye (to go in a circle)
            int costForMovingFromThisStation = cost[i];
            int gasEarnInNextStationj = gas[j];

            int currGas = gas[i] - costForMovingFromThisStation + gasEarnInNextStationj;

            while (j != i) {
                if (currGas < cost[j])
                    break;

                int costForMovingFromThisj = cost[j];
                j = (j + 1) % n;

                int gasEarnInNextStationj = gas[j];

                currGas = currGas - costForMovingFromThisj + gasEarnInNextStationj;
            }

            if (j == i)     // we completed full circle
                return i;
        }

        return -1;
    }
};
```

### 4.6 Line-by-line reasoning

| Line | Meaning / Why |
|---|---|
| `int n = gas.size();` | `gas` and `cost` always have the **same size**, so one `n` serves both. |
| `for (int i = 0; i < n; i++)` | The story's step 1: try **every** station as a candidate start. |
| `if (gas[i] < cost[i]) continue;` | Early rejection. If the gas you earn at station `i` is **less than** the cost of leaving station `i`, you can't even take the first step — this `i` is useless. What you earn must be **greater than or equal to** what you spend. (Note the life-lesson echo again.) |
| `int j = (i + 1) % n;` | Start the circular walk from the next station. The `% n` is what makes it circular. |
| `costForMovingFromThisStation = cost[i]` | Cost of leaving the *starting* station. |
| `gasEarnInNextStationj = gas[j]` | Gas collected on arriving at the next station. |
| `currGas = gas[i] - cost[i] + gas[j]` | The complete first hop in one expression: start with what station `i` gives, pay the exit toll, collect what station `j` gives. |
| `while (j != i)` | Keep circling until `j` wraps back to the start. If it does, the whole circle was covered. |
| `if (currGas < cost[j]) break;` | The **failure condition**: at station `j` you don't have enough gas to leave. No point continuing — abandon this `i`. |
| `j = (j + 1) % n;` | Move forward, circularly. |
| `currGas = currGas - costForMovingFromThisj + gasEarnInNextStationj;` | Same two-part hop: pay to leave the old `j`, collect at the new `j`. |
| `if (j == i) return i;` | We can exit the `while` in only **two** ways: (a) `j == i` — full circle done → `i` is the answer; (b) a `break` — failure. This `if` distinguishes the two. Since `break` only ever fires when `j != i`, this check is safe and unambiguous. |
| `return -1;` | The `for` loop finished, every index was tried, none completed the circle → impossible. |

**Note on the variable names:** the instructor deliberately uses long, descriptive names like `costForMovingFromThisStation` and `gasEarnInNextStationj`. His reason: *"This is nothing but just converting the story to code."* When the variable names read like the sentences of your explanation, the code writes itself and is far easier to explain to an interviewer.

*(Small C++ detail worth noticing: `gasEarnInNextStationj` is declared a second time **inside** the while loop, which shadows the outer one. It is used immediately after being assigned, so behaviour is correct — but in an interview you'd normally just reassign instead of redeclaring.)*

### 4.7 Complexity & result

- **Time:** **O(n²)** — for each of the `n` candidate starting points, we may walk up to `n` stations.
- **Space:** **O(1)**.
- **Submission result:** ❌ **Time Limit Exceeded — 34 / 37 test cases passed.**

The constraints on LeetCode are large, so O(n²) cannot pass. But as the instructor says: *"this is not something to cry about — building the brute force is itself a big achievement."* In a real interview, at this point the interviewer starts giving hints to push you toward greedy.

---

## 5. Approach 2 — Greedy (O(n))

### 5.1 Step 1: The feasibility check (this is where "आज का ज्ञान" pays off)

Recall the lesson: *total earning vs total expenditure*.

**Case 1 — our example:**

```text
gas  = [1, 2, 3, 4, 5]  →  total kamai  (earning) = 1 + 2 + 3 + 4 + 5 = 15
cost = [3, 4, 5, 1, 2]  →  total kharch (expense) = 3 + 4 + 5 + 1 + 2 = 15
```

Earning `15` = expense `15` → **feasible.** An answer definitely exists.

**Case 2 — a counter-example built in the lecture:**

```text
gas  = [2, 3, 4]  →  total earning  = 2 + 3 + 4 = 9
cost = [3, 4, 3]  →  total expense  = 3 + 4 + 3 = 10

9 < 10  →  IMPOSSIBLE
```

**Why this is airtight (the crucial reasoning):**

> No matter which index you start from — 0, 1, or 2 — a **full circular tour visits every single station**. So you will collect **every** `gas[i]` and you will pay **every** `cost[i]`. The totals are fixed and completely independent of your starting point; only the *order* changes.

Therefore, if `sum(gas) < sum(cost)`, the journey is short on fuel in an absolute sense, and **no ordering can save you**. Return `-1` immediately.

```cpp
if (totalKamai < totalKharcha)
    return -1;
```

### 5.2 Step 2: The guarantee (the part that makes the O(n) solution possible)

Now flip it:

> If **`totalKamai >= totalKharcha`**, then an answer is **guaranteed — 100%**. Some index definitely exists from which the full circle is completable.

This is the single most important consequence, because it means:

- After passing the feasibility check, we will **never** return `-1` again.
- We therefore **do not need to simulate the circular tour at all.** We don't have to verify a candidate by walking around. We only need to *find* the index — its validity is already guaranteed by the totals.
- That is precisely what collapses O(n²) into **O(n)**.

The instructor repeats this several times: *"main aankh band karke direct result return kar diya"* — "I return the result blindly; I never send `-1` from inside the loop, only from that one check at the top."

### 5.3 Step 3: The greedy scan

Maintain two variables:

- `total` — the running gas balance (profit/loss) for the current candidate journey. Starts at `0`.
- `result` — the currently assumed starting index. Starts at `0`.

For each `i`:

```text
total = total + gas[i] - cost[i];

if (total < 0) {
    total  = 0;      // reset the running balance
    result = i + 1;  // this candidate failed → try starting from the next index
}
```

**The greedy intuition (why this is called greedy):**

At every station you ask a single local question: *"am I in profit or in loss here?"*

- `gas[i] - cost[i] > 0` → this station **earns** you surplus.
- `gas[i] - cost[i] < 0` → this station puts you at a **loss**.

If the running balance ever dips below zero, the instructor's reaction is: *"I won't even try from here, I'll just move ahead."* You **greedily look for a profitable place to begin**, abandoning any prefix that has already gone into deficit, and you restart your accounting from scratch at the next index.

**Why `result = i + 1` and not something else:** if the running total went negative at station `i`, then the candidate start you were testing cannot work, and — importantly — no station between that candidate and `i` can work either (any such shorter journey would already have been stuck even sooner, since its prefix was non-negative before). So the earliest index still worth considering is `i + 1`, and we reset `total` to `0` because the journey is now being measured freshly from there.

**Why no circle check is needed at the end:** because of §5.2. If `total` stayed non-negative from `result` all the way to the end of the array, then combined with the guarantee that *some* valid start exists, `result` must be that start.

### 5.4 Full greedy dry run on the example

```text
gas  = [1, 2, 3, 4, 5]
cost = [3, 4, 5, 1, 2]

total  = 0
result = 0
```

| `i` | Computation | `total` after | `total < 0` ? | Action |
|---|---|---|---|---|
| 0 | `0 + 1 - 3` | **-2** | Yes | `total = 0`, `result = 1` |
| 1 | `0 + 2 - 4` | **-2** | Yes | `total = 0`, `result = 2` |
| 2 | `0 + 3 - 5` | **-2** | Yes | `total = 0`, `result = 3` |
| 3 | `0 + 4 - 1` | **3** | No | keep `result = 3` |
| 4 | `3 + 5 - 2` | **6** | No | keep `result = 3` |

**Final answer: `result = 3`** ✔ (matches the expected output)

**Narration of the trace (as in the lecture):**

- At `i = 0`: earn 1, spend 3 → `-2`. Going negative means station 0 cannot be the start. Reset and assume the answer is index 1.
- At `i = 1`: earn 2, spend 4 → `-2` again. Negative again. Reset, assume index 2.
- At `i = 2`: earn 3, spend 5 → `-2` again. Reset, assume index 3.
- **Don't panic at this point.** The instructor notes that you might get worried here — "we proved an answer is guaranteed (15 vs 15) but we still haven't found one!" Have patience; `i = 3` delivers.
- At `i = 3`: earn 4, spend 1 → `+3`. Finally positive! We have a backup of 3 units. Keep `result = 3`.
- At `i = 4`: `3 + 5 - 2 = 6`. Still comfortably positive — we never dipped negative after index 3.
- Loop ends. `result = 3`. **No wrap-around simulation required.**

Contrast this with the greedy view: at index 0 we saw `1 - 3 = -2` (loss → skip), index 1 `2 - 4 = -2` (loss → skip), index 2 `3 - 5 = -2` (loss → skip), index 3 `4 - 1 = +3` (**profit → start here, eyes closed**). *"Greedily I am looking for a profit; wherever I find profit, I start from there."*

### 5.5 Greedy pseudo-code (as written on the board)

```cpp
if (totalKamai < totalKharch)
    return -1;

total  = 0;
result = 0;

for (i = 0; i < n; i++) {
    total = total + gas[i] - cost[i];

    if (total < 0) {
        total  = 0;
        result = i + 1;
    }
}

return result;          // Time Complexity : O(n)
```

### 5.6 Final Greedy C++ solution (submitted)

```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int n = gas.size();

        int totalKamai  = accumulate(begin(gas),  end(gas),  0);
        int totalKharch = accumulate(begin(cost), end(cost), 0);

        if (totalKamai < totalKharch)
            return -1;

        int result_index = 0;
        int total        = 0;

        for (int i = 0; i < n; i++) {

            total += gas[i] - cost[i];

            if (total < 0) {
                result_index = i + 1;
                total = 0;
            }
        }

        return result_index;
    }
};
```

**Notes on the code:**

- `accumulate(begin(v), end(v), 0)` is the C++ STL way of summing a vector. The third argument `0` is the **initial value** of the sum. You could also write a manual `for` loop to compute both sums; `accumulate` is just cleaner.
- The `-1` is returned from **exactly one place** — the top-level feasibility check. Nowhere inside the loop. That is deliberate and is the direct consequence of the guarantee argument.
- `result_index` is initialised to `0`, i.e. "I assume index 0 is my answer until proven otherwise."

### 5.7 Complexity

| | Brute Force | Greedy |
|---|---|---|
| **Time** | O(n²) | **O(n)** (two passes for the sums + one pass for the scan) |
| **Space** | O(1) | **O(1)** |
| **Verdict** | TLE (34/37) | ✅ Accepted |

**Submission result:** `Accepted` · Runtime **94 ms** (beats 74.89%) · Memory **69.5 MB**.

---

## 6. Consolidated Summary & Interview Takeaways

### 6.1 The three ideas you must be able to state

1. **Movement rule:** arriving at station `i` gives `+gas[i]`; leaving station `i` costs `-cost[i]`. You're stuck whenever `currentGas < cost[i]`.
2. **Feasibility:** a full tour visits every station, so total gas and total cost are fixed regardless of the start. Hence `sum(gas) < sum(cost)` ⟹ `-1`, always. And `sum(gas) >= sum(cost)` ⟹ an answer **definitely** exists.
3. **Greedy scan:** track a running balance; the moment it goes negative at index `i`, discard everything so far, reset the balance to `0`, and set the candidate answer to `i + 1`. Whatever `result` holds at the end is the answer.

### 6.2 Interview-relevant points the instructor emphasised

- **Always present brute force first.** Interviewers (Google in particular) *expect* it and will then hint you toward the optimal. Jumping straight to the greedy trick without being able to explain the brute force is a weaker signal.
- **Turn the story into code.** Explain the algorithm as a plain narrative, then convert each sentence into a line. Use **meaningful, long variable names** so the code narrates itself.
- **The `% n` trick** is the standard way to traverse circularly — never forget it when the problem says "circular".
- **Greedy problems are genuinely hard to intuit.** The instructor admits there's no cleaner way he found to explain this one — it's "weird" by nature. That's normal; the pattern to internalise is *"skip any prefix that puts you in deficit, and restart from the next index."*

### 6.3 Edge cases & gotchas

- **Tank hitting exactly zero is fine.** The failure condition is `currentGas < cost[i]` (strictly less), so `5 - 5 = 0` still counts as a successful hop.
- **`gas[i] == cost[i]` at the start is acceptable** — earning must be *greater than or equal to* the spend, not strictly greater. That's why the brute force uses `if (gas[i] < cost[i]) continue;` and not `<=`.
- **`totalKamai == totalKharch` is feasible** (our main example is exactly this case, 15 vs 15) — the check is `<`, not `<=`.
- **Don't return `-1` from inside the greedy loop.** Once the sum check passes, `-1` is impossible by construction.
- **Don't confuse the answer with a gas value.** The output is the *starting index*, not how much fuel you end with.
- In the brute force, the `while` loop exits either via `j == i` (success) or via `break` (failure) — the trailing `if (j == i)` is what tells them apart, and it is unambiguous because `break` can only fire while `j != i`.
