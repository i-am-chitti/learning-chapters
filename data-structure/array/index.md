# Array

An array is:

- Contiguous memory
- Indexed access
- Predictable traversal cost

But more importantly, an array is a timeline.

You should mentally translate:

- indices → time / order
- values → state at that time

This is why arrays dominate:
- logs
- prices
- requests
- sensor data
- ratings
- metrics

## The ONLY 4 questions you should ask first

When you see any array problem, pause and ask:

**Q1. Do I need to look left, right, or both?**

- Left only → prefix thinking
- Right only → suffix thinking
- Both → two pointers / window

**Q2. Is order important?**

- Yes → cannot sort
- No → sorting / hashing allowed
- This single decision kills many wrong approaches early.

**Q3. Do I care about a range?**
If yes →
- sliding window
- prefix sums

**Q4. Am I tracking a best, count, or existence?**
- Best → max / min
- Count → frequency
- Existence → boolean / early exit

This tells you what state to maintain.

## Array problem patterns (core)

Almost all array problems fall into one of these:

| Pattern        | Mental Trigger               |
| -------------- | ---------------------------- |
| Single pass    | “Process once”               |
| Two pointers   | “Left + right interaction”   |
| Sliding window | “Continuous subarray”        |
| Prefix sum     | “Repeated range sum”         |
| State tracking | “Best so far / count so far” |

## Real backend analogy (important)

Imagine:

- Array = request latency per minute
- Index = minute
- Value = latency

Questions like:
- “Best window of 5 minutes?”
- “Did latency ever exceed X?”
- “How many bad minutes?”

These are array problems, not abstract puzzles.


## Problem

> Given an array of integers, find the maximum sum of any continuous subarray.

1. Do I care about order? - Order is important.
2. Do I care about a range? - Yes, A continuous subarray is literally a range: [i....j]. So, problem is "Find the best range" . Range problems → window / prefix / carry-forward thinking
3. Am I tracking best / count / existence? - Best - maximum.
4. Brute force optimization? - we are calculating sum again for j even though for i-1 already calculated

**Solution**

At every index i, ask only ONE question:

> Is it better to extend the previous subarray or start fresh here?

That’s it.
Everything else is noise.

```
current_sum = max(a[i], current_sum + a[i])
max_sum = max(max_sum, current_sum)
```

**Takeaway**

> At every step, make the smallest correct decision and carry only what helps.

## Kadane Algorithm

The pattern is:
- Maintain a rolling state
- Decide: continue vs restart
- Track best/worst

This symmetry reappears in:
- Max / Min product subarray
- Longest / shortest window
- Profit / loss streaks
- Error burst detection

## Sliding Window

### Why sliding window exists?

Sliding Window is born when:
- You care about continuous subarrays
- You care about a constraint (size, sum, count, condition)
- Brute force recomputes overlapping work

Think of it as:

> “A window that moves forward, never backward.”

That single rule gives you O(n).

### Types of sliding window

#### Type 1: Fixed-size window

window size known: `k`

Examples:
- Max sum of subarray of size k
- Average of last k values
- CPU usage in last k seconds

#### Type 2: Variable-size window

Window size changes based on a condition

Examples:
- Longest subarray with sum ≤ k
- Smallest subarray with sum ≥ k
- Longest substring without repeating characters

This is where real thinking begins.


### Fixed Window

#### Problem

> Given an array and k, find the maximum sum of any subarray of size k.

Brute force:
- For each window → sum again → O(nk)

Sliding window idea:
- Compute first window
- Slide by:
  - Remove left element
  - Add right element

### Variable Window

Core rule (never break this):

> Expand right → **fix violation** → shrink left

#### Problem

> Longest subarray with sum ≤ k (all positives)

Thinking:
- Expand right
- If sum > k → shrink left
- Track best length

#### Critical constraint

⚠️ This works only when numbers are non-negative.

Why?
- Removing elements always reduces sum
- **Monotonic behavior**

If negatives exist → sliding window breaks.

This is a very common interview trap.

#### Why sliding breaks for negative values?

Sliding window breaks because
- shrinking can increase the sum if a negative is removed
- expanding can decrease the sum if a negative is added

This means:

➡️ Window movement is no longer predictable

And that unpredictability is fatal to sliding window.

#### Principle

Sliding window requires monotonic behavior:
- Expand → sum moves in one direction
- Shrink → sum moves in the opposite direction

Negative numbers destroy monotonicity.

Once monotonicity is gone:
- Local fixes don’t guarantee global correctness
- “Fix violation by shrinking” becomes unsafe
- You can’t trust greedy boundary movement

> Sliding window depends on predictable state movement

> If window expansion or contraction can move the state in both directions, sliding window is invalid.

> Window validity can be restored by shrinking, and shrinking always moves the state toward validity.

> Sliding window is valid when fixing a violation is guaranteed to move the system closer to validity.