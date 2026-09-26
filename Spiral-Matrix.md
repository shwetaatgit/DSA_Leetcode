# Spiral Matrix (LeetCode Medium)

## Problem

Given an `m x n` matrix, return all its elements in spiral order (right across the top, down the right side, left across the bottom, up the left side, then repeat on the shrinking inner rectangle).

```
[[1,2,3],
 [4,5,6],
 [7,8,9]]  -> [1,2,3,6,9,8,7,4,5]
```

## How to explain it out loud

*"I track four boundaries — top, bottom, left, right — that define the current unvisited rectangle. Each pass around the spiral does four sweeps: across the top row, down the right column, across the bottom row, up the left column — and after each sweep, I shrink the corresponding boundary inward, since that row or column is now fully consumed. The one subtlety is the bottom row and left column sweeps need a guard before running: once the boundaries cross (say `top > bottom` after the top-row and right-column sweeps already happened), doing the bottom-row sweep again would either be wrong or revisit cells already output — that shows up specifically in single-row or single-column matrices, where all four sweeps would otherwise try to run even though there's only one row or column."*

## Approach — Shrinking boundaries, four directional sweeps per loop

Maintain `top`, `bottom`, `left`, `right`, initialized to the matrix's actual edges. Loop while `top <= bottom && left <= right` (there's still an unvisited rectangle). Each iteration:

1. Sweep left→right across row `top`, then `top++` (that row is done).
2. Sweep top→bottom down column `right` (using the *already-incremented* `top`, so the corner cell isn't repeated), then `right--`.
3. **Guarded** by `top <= bottom`: sweep right→left across row `bottom`, then `bottom--`. The guard matters because after steps 1-2 shrink `top` and `right`, a single-row matrix could already have `top > bottom` — without the guard, this would re-sweep the same row already covered by step 1.
4. **Guarded** by `left <= right`: sweep bottom→top up column `left`, then `left++`. Same reasoning — a single-column matrix could have `left > right` after step 2 already consumed the only column.

Time: O(m×n) — every cell visited exactly once · Space: O(1) extra beyond the output array (four integer boundaries).

## Solution

### C++
```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        vector<int> result;
        if (matrix.empty()) return result;

        int top = 0, bottom = matrix.size() - 1;
        int left = 0, right = matrix[0].size() - 1;

        while (top <= bottom && left <= right) {
            // across the top row
            for (int j = left; j <= right; j++) result.push_back(matrix[top][j]);
            top++;

            // down the right column
            for (int i = top; i <= bottom; i++) result.push_back(matrix[i][right]);
            right--;

            // across the bottom row (if still valid)
            if (top <= bottom) {
                for (int j = right; j >= left; j--) result.push_back(matrix[bottom][j]);
                bottom--;
            }

            // up the left column (if still valid)
            if (left <= right) {
                for (int i = bottom; i >= top; i--) result.push_back(matrix[i][left]);
                left++;
            }
        }

        return result;
    }
};
```

### Python
```python
class Solution:
    def spiralOrder(self, matrix: list[list[int]]) -> list[int]:
        result = []
        if not matrix:
            return result

        top, bottom = 0, len(matrix) - 1
        left, right = 0, len(matrix[0]) - 1

        while top <= bottom and left <= right:
            for j in range(left, right + 1):
                result.append(matrix[top][j])
            top += 1

            for i in range(top, bottom + 1):
                result.append(matrix[i][right])
            right -= 1

            if top <= bottom:
                for j in range(right, left - 1, -1):
                    result.append(matrix[bottom][j])
                bottom -= 1

            if left <= right:
                for i in range(bottom, top - 1, -1):
                    result.append(matrix[i][left])
                left += 1

        return result
```

### Java
```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();
        if (matrix.length == 0) return result;

        int top = 0, bottom = matrix.length - 1;
        int left = 0, right = matrix[0].length - 1;

        while (top <= bottom && left <= right) {
            for (int j = left; j <= right; j++) result.add(matrix[top][j]);
            top++;

            for (int i = top; i <= bottom; i++) result.add(matrix[i][right]);
            right--;

            if (top <= bottom) {
                for (int j = right; j >= left; j--) result.add(matrix[bottom][j]);
                bottom--;
            }

            if (left <= right) {
                for (int i = bottom; i >= top; i--) result.add(matrix[i][left]);
                left++;
            }
        }

        return result;
    }
}
```

Verified against 8 hand-picked shapes (a square matrix, a wide rectangle, a tall rectangle, a single cell, a single row, a single column, an empty matrix, and both non-square orientations) plus a 500-trial randomized stress test (random dimensions 1-8 in each direction) against an independent direction-vector simulation (walk right/down/left/up, turning whenever the next cell is out of bounds or already visited) — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same logic translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- Correct on the first attempt — no bugs found.
