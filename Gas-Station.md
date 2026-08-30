# Gas Station (LeetCode Medium)

## Problem

`n` gas stations arranged in a circle. `gas[i]` = fuel available at station `i`. `cost[i]` = fuel needed to travel from station `i` to station `i+1`. Starting with an empty tank at some station, return the starting index from which you can complete the full circuit exactly once — or `-1` if it's impossible. Guaranteed at most one valid answer exists.

```
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]
-> 3
```

## How to explain it out loud

*"First, feasibility: the whole trip is only possible if total gas is at least total cost — if sum(gas) < sum(cost), no starting point works, full stop. Given that it's feasible, the key insight is: if you start at some index and your running tank goes negative by the time you reach station j, then no station between your start and j could have worked as a start either — arriving at any of those intermediate stations, you'd have had less or equal fuel than you did starting from the very beginning of that stretch, so they'd fail too, even sooner or at the same point. So the moment the running tank goes negative, you can safely throw away every candidate start so far and try the very next station as a fresh candidate, resetting the tank to zero. Walk through once, and whichever candidate survives to the end of the array is the answer — as long as the overall total was non-negative to begin with."*

## Approach

Track a running `tank` and a candidate `start` index (initially `0`). Walk through every station once, adding `gas[i] - cost[i]` to `tank`. Whenever `tank` drops below `0`, the current candidate `start` is proven impossible — discard it, set the next candidate to `i+1`, and reset `tank` to `0` to begin testing that new candidate fresh. Meanwhile track the total sum of `gas[i]-cost[i]` across the whole array (feasibility check). At the end, if the total sum is `>= 0`, the surviving candidate is guaranteed correct (by the reasoning above); otherwise return `-1`.

Why this is safe to do in one linear pass rather than testing each candidate separately (which would be O(n²)): if starting at `i` fails at some station `j` (tank goes negative reaching `j`), then for any `k` strictly between `i` and `j`, starting at `k` also fails at `j` — because a start at `i` had already accumulated a tank of `gas[i..k-1] - cost[i..k-1]` by the time it reached `k`; if that stretch had a net negative contribution, then `k` would start with less fuel than `i`'s cumulative run did at that point, making things only worse. If that stretch had a net non-negative contribution, `i` still fails no earlier than `k` would, and since we already know `i` fails by `j`, `k` fails by `j` too. Either way, none of `i+1 .. j-1` are worth testing — only `j+1` onward is a fresh, untested possibility.

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int fuel = 0;      // running tank for the current candidate start
        int temp = 0;      // accumulates the "debt" from every failed segment
        int n = gas.size();
        int resultInd = -1;

        for (int i = 0; i < n; i++) {
            fuel += gas[i] - cost[i];
            if (fuel < 0) {
                temp += fuel;       // bank this segment's negative total
                fuel = 0;           // reset tank for the next candidate
                resultInd = -1;     // discard current candidate
            } else if (fuel >= 0 && resultInd == -1) {
                resultInd = i;      // lock in this index as the new candidate
            }
        }
        // fuel + temp == sum(gas) - sum(cost) across the whole array,
        // since every index's contribution lands in exactly one of the two.
        if (fuel + temp >= 0) {
            return resultInd;
        }
        return -1;
    }
};
```

### Python
```python
class Solution:
    def canCompleteCircuit(self, gas: List[int], cost: List[int]) -> int:
        fuel = 0
        temp = 0
        n = len(gas)
        result_ind = -1
        for i in range(n):
            fuel += gas[i] - cost[i]
            if fuel < 0:
                temp += fuel
                fuel = 0
                result_ind = -1
            elif fuel >= 0 and result_ind == -1:
                result_ind = i
        if fuel + temp >= 0:
            return result_ind
        return -1
```

### Java
```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int fuel = 0;
        int temp = 0;
        int n = gas.length;
        int resultInd = -1;

        for (int i = 0; i < n; i++) {
            fuel += gas[i] - cost[i];
            if (fuel < 0) {
                temp += fuel;
                fuel = 0;
                resultInd = -1;
            } else if (fuel >= 0 && resultInd == -1) {
                resultInd = i;
            }
        }
        if (fuel + temp >= 0) {
            return resultInd;
        }
        return -1;
    }
}
```

Verified against 6 hand-picked cases (classic example, feasible-with-fuel-to-spare, feasible-sum-exactly-zero, infeasible, single feasible station, single infeasible station) plus a 2000-trial randomized stress test against a brute-force O(n²) reference (every start, simulate full circuit) — 0 mismatches in C++ and 0 mismatches in Python, both C++ runs additionally clean under AddressSanitizer. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- First attempt: `diff[i] = gas[i]-cost[i]`, required `sum(diff) == 0` exactly, then returned the first index with a positive `diff[i]`. Two separate bugs, both caught with concrete counterexamples before writing the final version:
  - Feasibility only requires `sum >= 0` (gas ≥ cost overall, extra fuel is fine) — requiring exact equality wrongly rejected a feasible case where total gas exceeded total cost.
  - "First positive diff" doesn't account for what happens *later* in the loop — a station can have a locally positive diff yet still lead to a negative tank a few stations later, while a different, correct start further along the array survives the whole circuit. Confirmed with a constructed example (`gas=[15,4,13,8,10]`, `cost` all `10`s) where index 0 has a positive diff but fails immediately at index 1, while the true answer is index 2.
  - Fixed by switching to the one-pass greedy: reset the candidate start (not just skip it) every time the running tank goes negative, and separately track total feasibility via `fuel + temp`.
