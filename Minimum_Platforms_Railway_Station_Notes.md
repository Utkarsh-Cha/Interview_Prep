# Minimum Number of Platforms Required for a Railway Station
### (Greedy Algorithms Playlist — Detailed Study Notes)

---

## 1. Problem Statement

You are given `N` trains. For **every** train you are given two pieces of information:

- **Arrival time** — the time the train reaches the station
- **Departure time** — the time the train leaves the station

These are given as two separate arrays, where index `i` of both arrays refers to the **same** train:

```text
arr[] = [ 900   945   955   1100   1500   1800 ]
dep[] = [ 920  1200  1130   1150   1900   2000 ]
        train1 train2 train3 train4 train5 train6
```

So train 1 arrives at 9:00 and leaves at 9:20, train 2 arrives at 9:45 and leaves at 12:00, and so on. Times are written in 24-hour `HHMM` format (`1500` = 3:00 PM, `1900` = 7:00 PM).

**Task:** Find the **minimum** number of platforms the station must have so that **all `N` trains** can arrive and depart without any train ever having to wait.

The constraint that makes this non-trivial: **one platform can hold only one train at a time.** A platform becomes free only after the train standing on it departs.

> The word "minimum" is the key word in the question. You are not asked "does this work?" — you are asked for the smallest platform count that still works.

---

## 2. Building Intuition — Assigning Trains to Platforms by Hand

The instructor first solves the example manually, train by train, exactly the way a station master would. Follow this carefully, because the whole solution falls out of this walkthrough.

### Step-by-step assignment

**Train 1 → arrives 900, departs 920**
This is the very first train. There is nothing else at the station, so we obviously need at least one platform.
→ **Platform 1: 900 – 920**

**Train 2 → arrives 945, departs 1200**
Do we need a *second* platform? Let's check. Train 1 left the station at **920**. Train 2 arrives at **945**, which is *after* 920. So platform 1 is already empty and lying free by then.
→ Reuse **Platform 1: 945 – 1200**

**Train 3 → arrives 955, departs 1130**
Can it go to platform 1? **No.** Platform 1 is occupied by train 2 from 945 all the way to 1200, and 955 lies inside that window. So platform 1 is busy.
→ We are forced to open **Platform 2: 955 – 1130**

**Train 4 → arrives 1100, departs 1150**
- Platform 1? Busy (train 2 is there until 1200). ✗
- Platform 2? Busy (train 3 is there until 1130). ✗
→ We are forced to open **Platform 3: 1100 – 1150**

**Train 5 → arrives 1500, departs 1900**
By 1500, every train so far has departed (latest departure so far was 1200). **All three platforms are empty**, so we can send it to any of them. Pick platform 1.
→ **Platform 1: 1500 – 1900**

**Train 6 → arrives 1800, departs 2000**
- Platform 1? Busy — train 5 stays until 1900, and 1800 < 1900. ✗
- Platform 2? Free. ✓
→ **Platform 2: 1800 – 2000**

### Final platform picture

| Platform | Trains parked on it |
|---|---|
| Platform 1 | 900–920, then 945–1200, then 1500–1900 |
| Platform 2 | 955–1130, then 1800–2000 |
| Platform 3 | 1100–1150 |

**Answer for this example = 3 platforms.**

### Why exactly 3, and not 4?

Four platforms would also work — but the fourth platform would simply never be used by anybody. It would be **overkill**. The question asks for the *minimum*, i.e. the smallest number that still lets every train be accommodated. Here 3 is the smallest such number, because at one moment in the day (around 11:00–11:30) there were genuinely **3 trains standing at the station simultaneously**, and you cannot squeeze 3 simultaneous trains onto 2 platforms.

---

## 3. THE CORE INSIGHT — Reduce the problem to "maximum overlap"

This is the single most important idea in the whole lecture:

> **The minimum number of platforms required = the maximum number of trains that are present at the station at the same instant = the maximum number of `[arrival, departure]` intervals that intersect at any single point in time.**

Why is this true?

- **Lower bound (why you need at least that many):** if at some instant `k` trains are simultaneously at the station, each of them needs its own platform at that instant, so you need **at least** `k` platforms.
- **Upper bound (why that many is enough):** at any instant, the number of occupied platforms is exactly the number of trains currently present. If that number never exceeds `k`, then `k` platforms are always sufficient — a train never fails to find a free platform.

So the whole problem collapses into: **find the maximum number of intersecting intervals.**

In our example, look at these three trains:

```text
 9:45 ------------------------------------------ 12:00     (train 2)
        9:55 ------------------ 11:30                        (train 3)
                 11:00 ------------ 11:50                    (train 4)
```

All three overlap in the window 11:00–11:30. That is the worst moment of the day, and that is why the answer is 3.

---

## 4. Brute Force Approach

### 4.1 Idea

Since the answer is "maximum number of intersecting intervals", the most naive thing we can do is:

> Take every train one by one. For that train, count how many other trains intersect with it. Keep the maximum such count over all trains.

For each train, start the count at **1** (the train itself always occupies one platform), then add 1 for every other train that overlaps with it.

### 4.2 Dry run on the example

```text
arr[] = [ 900   945   955  1100  1500  1800 ]
dep[] = [ 920  1200  1130  1150  1900  2000 ]
```

| Train (interval) | Intersects with | cnt |
|---|---|---|
| Train 1 `[900, 920]` | nobody (everything else starts at 945 or later) | 1 |
| Train 2 `[945, 1200]` | Train 3 `[955,1130]`, Train 4 `[1100,1150]` | 3 |
| Train 3 `[955, 1130]` | Train 2, Train 4 | 3 |
| Train 4 `[1100, 1150]` | Train 2, Train 3 | 3 |
| Train 5 `[1500, 1900]` | Train 6 `[1800,2000]` | 2 |
| Train 6 `[1800, 2000]` | Train 5 | 2 |

Maximum `cnt` seen = **3** → answer is **3**. This matches the manual platform assignment. ✔

(For train 1, the instructor puts a red ✗ under every other pair — nothing overlaps with 900–920. For train 2, checkmarks appear under `[955,1130]` and `[1100,1150]`.)

### 4.3 The 4 intersection cases (important — the interviewer may ask this)

Suppose we are checking train `i` against train `j`. Draw train `i` as a segment with two dots (arrival dot and departure dot). Another train `j` can overlap with it in **four distinct ways**:

```text
Train i:        arr_i •-----------------------• dep_i
```

| Case | Description | Picture |
|---|---|---|
| **1** | `j` arrives **before** `i` arrives, and departs **after** `i` departs (`j` completely covers `i`) | `•---------------------------•` (wider on both sides) |
| **2** | `j` arrives **before** `i` arrives, and departs **in the middle** of `i`'s stay | `•--------•` (starts left, ends inside) |
| **3** | `j` arrives **in the middle** of `i`'s stay, and departs **after** `i` departs | `      •--------•` (starts inside, ends right) |
| **4** | `j` arrives **and** departs entirely **inside** `i`'s stay (`i` completely covers `j`) | `   •-----•` (fully inside) |

Summarised in the instructor's own words: *arriving before & departing later; arriving before & departing middle; arriving middle & departing later; arriving middle & departing middle.*

All four must be handled, otherwise you will miss some overlaps.

**Useful shortcut worth knowing:** all four cases collapse into one clean condition. Two intervals `[a_i, d_i]` and `[a_j, d_j]` intersect **iff**

```text
a_i <= d_j  AND  a_j <= d_i
```

Equivalently, they do **not** intersect iff one finishes strictly before the other starts. Writing it this way is far less error-prone than writing four separate `if` conditions.

### 4.4 Brute force code (as written on screen)

```text
func( arr, dep )
{
    maxCnt = 0

    for( i = 0 -> n-1 )
    {
        cnt = 1                      // the train itself needs 1 platform

        for( j = i+1 -> n-1 )
        {
            if ( /* arr[i], dep[i]  intersects with  arr[j], dep[j] */ )
            {
                cnt++;
            }
            maxCnt = max(maxCnt, cnt)
        }
    }

    return maxCnt;
}
```

**Line-by-line logic**

- `maxCnt = 0` — global best answer across all trains.
- Outer loop `i` — fixes the "reference" train whose overlaps we are counting.
- `cnt = 1` — **must be reset inside the outer loop**, because every new reference train starts fresh, needing its own single platform.
- Inner loop `j = i+1` — compares the reference train against the remaining trains.
- The `if` condition is the intersection check between `(arr[i], dep[i])` and `(arr[j], dep[j])` — this is where the 4 cases (or the single combined condition) go.
- `maxCnt = max(maxCnt, cnt)` — track the running best.
- `return maxCnt` — the maximum simultaneous overlap = minimum platforms.

### 4.5 Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N²)` | Outer loop `O(N)` × inner loop `O(N)` |
| **Space** | `O(1)` | No extra data structure is used |

### 4.6 Why the interviewer will reject this

`O(N²)` is **quadratic**. For large `N` this is too slow, and no interviewer will be satisfied with it. This is the point where you will be asked to optimise.

> **Reading the hint in the complexity:** once `O(N²)` is rejected, the target is almost always `O(N log N)` or `O(N)`. `O(N log N)` is a strong signal that **sorting** is involved. Keep that in mind — it is exactly where the optimal solution goes.

### 4.7 Caveat on this brute force (good to be aware of)

The instructor himself calls this the "extreme naive solution that comes to my brain" and says *"you can definitely optimize slightly here and there."*

Be aware of one subtlety: counting "how many trains overlap with train `i`" is not perfectly identical to "how many trains are present simultaneously", because two trains can each overlap train `i` at *different* moments without overlapping each other. For example `i = [10,20]`, `j₁ = [5,11]`, `j₂ = [19,25]`: all pairwise checks against `i` succeed giving `cnt = 3`, but at no single instant are 3 trains present (the true answer is 2). So the pairwise-count brute force can **overcount** on such inputs. This does not affect the given example, and the optimal approach below does not suffer from this issue at all — it counts the true simultaneous occupancy. Mention this only if pushed; the important takeaway is that the **event-based optimal solution is the correct and safe one**.

---

## 5. Optimal Approach — Intuition ("Stand Outside the Station and Watch the Clock")

### 5.1 The real-world picture

Forget arrays for a moment. Imagine you physically go and stand outside the railway station in the morning, with a notebook, and simply **watch what happens as the clock ticks forward**. Every time *something happens* (a train arrives, or a train departs), you note it down.

Here is the whole day of our example, in chronological order:

| Time | Event | Which train |
|---|---|---|
| 900 | Arrival | Train 1 |
| 920 | Departure | Train 1 |
| 945 | Arrival | Train 2 |
| 955 | Arrival | Train 3 |
| 1100 | Arrival | Train 4 |
| 1130 | Departure | Train 3 |
| 1150 | Departure | Train 4 |
| 1200 | Departure | Train 2 |
| 1500 | Arrival | Train 5 |
| 1800 | Arrival | Train 6 |
| 1900 | Departure | Train 5 |
| 2000 | Departure | Train 6 |

Notice the instructor explicitly pausing at one spot: after 11:00 the next event is **11:30, not 12:00** — even though train 2 (which departs at 1200) arrived *before* train 3 (which departs at 1130). Events are ordered purely by **time**, not by train number.

### 5.2 Maintain a running counter

Keep a variable `cnt` = number of platforms currently in use. Start at `cnt = 0` (when you arrive at the station, nothing is happening).

- On an **arrival** → a platform gets occupied → `cnt = cnt + 1`
- On a **departure** → a platform gets released → `cnt = cnt - 1`

Walk through the timeline:

```text
Event:  (900,A) (920,D) (945,A) (955,A) (1100,A) (1130,D) (1150,D) (1200,D) (1500,A) (1800,A) (1900,D) (2000,D)
cnt:  0    1       0       1       2        3        2        1        0        1        2        1        0
                                            ▲
                                        max = 3
```

The highest value `cnt` ever reaches is **3**. That is the answer — the same 3 we got by hand and by brute force. ✔

**Why this works:** `cnt` at any moment is literally the number of trains standing at the station at that moment. The peak of `cnt` is therefore the maximum simultaneous overlap, which (from Section 3) is exactly the minimum number of platforms.

### 5.3 Approach A — merge into a third array (uses extra space)

The direct implementation of the above:

1. Create a third array containing **all** events: each arrival tagged `A`, each departure tagged `D`.
2. **Sort this combined array by time.**
3. Sweep left to right, doing `cnt++` on `A` and `cnt--` on `D`, tracking `maxCnt`.

This works perfectly, but it costs **`O(N)` extra space** for the merged array.

### 5.4 The clever observation — sort `arr[]` and `dep[]` independently

Can we avoid the third array? Yes.

Ask: **what do we actually need?** We only need the **times, in sorted order**, along with the knowledge of whether each time is an arrival or a departure. We do **not** need to know *which* train an arrival belongs to, and we do **not** need arrival and departure of the same train to stay paired together — because we are only keeping a **count**, not tracking individual trains onto specific platforms.

> This is the mental unlock: *"I'm not concerned about arrival and departure staying together, because as the day passes by I'm just keeping a count."*

So we can simply:
- Sort `arr[]` on its own → the arrival times in chronological order
- Sort `dep[]` on its own → the departure times in chronological order

The pairing between `arr[i]` and `dep[i]` is destroyed, and that is **fine**. The *multiset* of arrival times is unchanged, and the *multiset* of departure times is unchanged. Therefore, for any time `t`, "number of arrivals up to `t`" and "number of departures up to `t`" are unchanged, and so the occupancy count `arrivals(t) − departures(t)` at every moment is unchanged too. The peak is preserved.

For our example:

```text
arr[] = [ 900   945   955  1100  1500  1800 ]     ← already sorted
dep[] = [ 920  1200  1130  1150  1900  2000 ]     ← NOT sorted

after sorting dep[]:
dep[] = [ 920  1130  1150  1200  1900  2000 ]
```

Now instead of physically merging, we **merge them logically using two pointers** — exactly like the merge step of merge sort.

---

## 6. Optimal Algorithm — Two Pointers

### 6.1 The algorithm

- `i` points into the sorted `arr[]`, `j` points into the sorted `dep[]`. Both start at 0.
- `cnt = 0` (platforms currently in use), `maxCnt = 0` (best answer).
- Repeatedly ask: **which event happens next in time — the next arrival or the next departure?**
  - If `arr[i] <= dep[j]` → an **arrival** happens first → `cnt++`, `i++`
  - Else → a **departure** happens first → `cnt--`, `j++`
- After each event, update `maxCnt = max(maxCnt, cnt)`.
- Loop while `i < N`.

### 6.2 Why the loop condition is `while (i < N)` and not something else

Because **arrivals always get exhausted first**. Every train must arrive before it departs, so at any point `i >= j`, and `arr` runs out before `dep` does.

More importantly, once **all trains have arrived**, no new platform can ever be occupied again — from that point on, `cnt` can only go *down*. So there is no chance of finding a new maximum after `arr[]` is exhausted, and we can stop the moment `i` reaches `N`. Continuing to drain `dep[]` would be pure waste.

### 6.3 Full dry run on the example

```text
arr[] = [ 900   945   955  1100  1500  1800 ]
dep[] = [ 920  1130  1150  1200  1900  2000 ]
```

| Step | `arr[i]` | `dep[j]` | `arr[i] <= dep[j]`? | Action | `cnt` | `maxCnt` |
|---|---|---|---|---|---|---|
| start | — | — | — | — | 0 | 0 |
| 1 | 900 | 920 | ✔ | arrival → `cnt++`, `i++` | 1 | 1 |
| 2 | 945 | 920 | ✘ | departure → `cnt--`, `j++` | 0 | 1 |
| 3 | 945 | 1130 | ✔ | arrival → `cnt++`, `i++` | 1 | 1 |
| 4 | 955 | 1130 | ✔ | arrival → `cnt++`, `i++` | 2 | 2 |
| 5 | 1100 | 1130 | ✔ | arrival → `cnt++`, `i++` | **3** | **3** |
| 6 | 1500 | 1130 | ✘ | departure → `cnt--`, `j++` | 2 | 3 |
| 7 | 1500 | 1150 | ✘ | departure → `cnt--`, `j++` | 1 | 3 |
| 8 | 1500 | 1200 | ✘ | departure → `cnt--`, `j++` | 0 | 3 |
| 9 | 1500 | 1900 | ✔ | arrival → `cnt++`, `i++` | 1 | 3 |
| 10 | 1800 | 1900 | ✔ | arrival → `cnt++`, `i++` (`i` = 6) | 2 | 3 |
| end | `i == N` → stop | | | | | **return 3** |

Count sequence produced: `0 → 1 → 0 → 1 → 2 → 3 → 2 → 1 → 0 → 1 → 2`, peak **3** — exactly the sequence written on the board.

**Answer = 3 platforms.** ✔ (Matches both the manual assignment and the brute force.)

Note how the two-pointer walk reproduces the real-world event timeline (900 A, 920 D, 945 A, 955 A, 1100 A, 1130 D, …) **without ever building the merged array.**

### 6.4 Optimal code (as written on screen)

```text
func( arr, dep )
{
    sort(arr);      sort(dep);
    i = 0;          j = 0;
    cnt = 0;        maxCnt = 0;

    while( i < N )
    {
        if( arr[i] <= dep[j] )      // next event is an ARRIVAL
        {
            cnt = cnt + 1;
            i   = i + 1;
        }
        else                        // next event is a DEPARTURE
        {
            cnt = cnt - 1;
            j   = j + 1;
        }

        maxCnt = max(maxCnt, cnt);
    }

    return maxCnt;
}
```

**Execution flow in words**

1. Sort both arrays independently (you write the sort call according to your language — `sort(arr.begin(), arr.end())` in C++, `Arrays.sort(arr)` in Java, `arr.sort()` in Python).
2. Two pointers `i` (arrivals) and `j` (departures), both starting at 0.
3. `cnt` tracks live occupancy; `maxCnt` remembers the peak.
4. Each iteration processes **exactly one event**, choosing whichever is earlier in time.
5. `maxCnt` is updated after **every** event (placing it outside the if-else means it is never missed).
6. Return `maxCnt`.

### 6.5 The `<=` sign — an important edge case

The condition is `arr[i] <= dep[j]`, **not** `arr[i] < dep[j]`.

This means: if a train arrives at exactly the same time as another train departs, we treat the arrival as happening **first**, so the count goes up before it comes down — i.e. the two trains are considered to need **two separate platforms**. This is the standard interpretation for this problem: a platform is not instantly available at the same clock value; the arriving train cannot occupy a platform in the very same minute another is leaving it.

If an interviewer explicitly states that a platform freed at time `t` *can* be reused by a train arriving at time `t`, you would flip the condition to `arr[i] < dep[j]`. **Clarify this with the interviewer** — it is a classic ambiguity in this question.

### 6.6 Complexity analysis

**Time:**
- Sorting `arr[]` → `O(N log N)`
- Sorting `dep[]` → `O(N log N)`
- The two-pointer while loop → each iteration advances **either** `i` **or** `j` by one. Across the whole loop we walk through both arrays, so the traversal is `O(2N)` in the worst case (not `O(N)` — you traverse *two* arrays, sometimes stepping in the first, sometimes in the second).

Total, as written on the board:

```text
TC → 2(N log N + N)   ≡   O(N log N)
```

**Space:**

```text
SC → O(1)
```

No extra array is allocated — the sorting is done **in place** on the given arrays.

### 6.7 Warning — you are distorting the input arrays

The `O(1)` space comes at a price: this solution **destroys the original arrays** by sorting them in place, and in particular it destroys the `arr[i] ↔ dep[i]` pairing.

If the interviewer says *"you are not allowed to modify / distort the given arrays"*, then fall back to the third-array approach (Section 5.3): copy all arrivals and departures with tags into a new array, sort it by time, and sweep. The **time complexity stays essentially the same** (`O(N log N)`); only the space complexity becomes `O(N)`.

Always ask/state this trade-off in an interview — it shows you understand the cost of in-place mutation.

---

## 7. Comparison of Approaches

| Approach | Core idea | Time | Space | Notes |
|---|---|---|---|---|
| **Brute force** | For each train, count how many trains intersect it; take the max | `O(N²)` | `O(1)` | Needs the 4 intersection cases; rejected by interviewers; can overcount (see 4.7) |
| **Optimal (two pointers, in place)** | Sort `arr` and `dep` separately, merge-walk by time, `+1` on arrival / `−1` on departure, track peak | `2(N log N + N)` → `O(N log N)` | `O(1)` | Distorts the input arrays |
| **Optimal (merged third array)** | Put all events in one array with A/D tags, sort by time, sweep | `O(N log N)` | `O(N)` | Use when the input must not be modified |

---

## 8. Quick Revision Sheet

1. **Reframe the question:** minimum platforms = **maximum number of trains present at the station at the same instant** = maximum overlapping intervals.
2. **Brute force:** double loop, count pairwise intersections, 4 overlap cases, `O(N²)` time / `O(1)` space. Too slow.
3. **`O(N²)` rejected ⇒ aim for `O(N log N)` ⇒ think sorting.**
4. **Optimal intuition:** stand outside the station, watch the clock. Every arrival takes a platform (`+1`), every departure frees one (`−1`). The peak of the running count is the answer.
5. **Key trick:** you may sort `arr[]` and `dep[]` **independently** — the arrival/departure pairing does not matter because you are only maintaining a *count*, not tracking specific trains.
6. **Two-pointer merge:** `if (arr[i] <= dep[j]) { cnt++; i++; } else { cnt--; j++; }`, update `maxCnt` every step, loop `while (i < N)`.
7. **Why `while (i < N)`:** once all trains have arrived, the count can only decrease, so no new maximum is possible.
8. **`<=` not `<`:** an arrival at the same clock time as a departure needs its own platform (clarify with the interviewer).
9. **Complexity:** `2(N log N + N)` time, `O(1)` space — the `2N` is because the walk covers *both* arrays.
10. **Caveat to volunteer:** the in-place version distorts the input; if that is disallowed, use a merged third array with `O(N)` space.
11. **Worked example answer:** `arr = [900,945,955,1100,1500,1800]`, `dep = [920,1200,1130,1150,1900,2000]` → **3 platforms**, because trains 2, 3 and 4 all overlap between 11:00 and 11:30.
