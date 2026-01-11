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