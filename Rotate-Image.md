# Rotate Image (LeetCode Medium)

## Problem

Given an `n x n` 2D matrix representing an image, rotate it 90 degrees clockwise, **in place**.

```
matrix = [[1,2,3],
          [4,5,6],
          [7,8,9]]
->
        [[7,4,1],
         [8,5,2],
         [9,6,3]]
```

## How to explain it out loud

*"Two simple operations combined. First, transpose the matrix — swap matrix[i][j] with matrix[j][i] across the main diagonal, which flips rows into columns. If you compare the transposed matrix to the target rotated matrix row by row, each row has the right values but in reversed left-right order. So the second step is: reverse every row. Transpose turns rows into columns, and reversing each row then flips the left-right order within each of those new rows — together that's exactly a 90-degree clockwise rotation, and both steps can be done in place with no extra matrix needed."*

## Approach

**Transpose in place:** for every pair `i < j`, swap `matrix[i][j]` with `matrix[j][i]`. This only needs the upper triangle (`j` starting at `i+1`) since each swap handles both symmetric positions at once.

**Reverse each row:** after the transpose, reverse the elements within every individual row.

Together, these two operations produce a 90-degree clockwise rotation with no auxiliary matrix — the transpose handles turning columns into rows, and the row-reversal handles getting the left-right order correct within each new row.

Time: O(n²) (visit every cell a constant number of times) · Space: O(1) extra, fully in place

## Solution

### C++ (using `std::swap` / `std::reverse`)
```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        // Transpose: swap elements across the main diagonal
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                std::swap(matrix[i][j], matrix[j][i]);
            }
        }

        // Reverse each row
        for (int i = 0; i < n; ++i) {
            reverse(matrix[i].begin(), matrix[i].end());
        }
    }
};
```

### C++ (manual swaps, no standard library helpers)
```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        // Transpose: swap elements across the main diagonal, by hand
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }

        // Reverse each row with a manual two-pointer swap (no std::reverse)
        for (int i = 0; i < n; i++) {
            int l = 0, r = n - 1;
            while (l < r) {
                int temp = matrix[i][l];
                matrix[i][l] = matrix[i][r];
                matrix[i][r] = temp;
                l++;
                r--;
            }
        }
    }
};
```
Same two-step idea (transpose, then reverse each row), just without relying on `std::swap` or `std::reverse` — the transpose uses a manual temp-variable swap, and each row reversal uses the same two-pointer inward-swap pattern used earlier in this marathon (e.g. Reverse Words in a String, Valid Palindrome).

### Python
```python
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        n = len(matrix)
        for i in range(n):
            for j in range(i+1, n):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
        for row in matrix:
            row.reverse()
```

### Java
```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
        for (int[] row : matrix) {
            int l = 0, r = row.length - 1;
            while (l < r) {
                int temp = row[l];
                row[l] = row[r];
                row[r] = temp;
                l++;
                r--;
            }
        }
    }
}
```

Verified against 4 hand-picked cases (the classic 3×3 example, a 4×4 case, a single-element matrix, a 2×2 matrix) plus a 200-trial randomized stress test against an independent reference implementation (builds a fresh rotated matrix directly from the position formula `res[j][n-1-i] = matrix[i][j]`, rather than transpose+reverse) — 0 mismatches, for both the standard-library version and the manual-swap version. Both C++ versions clean under AddressSanitizer. Python matches on all 4 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — correct on the first attempt, including the conceptual leap of decomposing the rotation into transpose + row-reversal.
