# Zigzag Conversion (LeetCode Medium)

## Problem

Given a string `s` and an integer `numRows`, arrange the characters in a zigzag pattern across `numRows` rows (down one column, then diagonally up to the top, then down again, repeating), then read the result back row by row, left to right.

```
s = "PAYPALISHIRING", numRows = 3

P   A   H   N
A P L S I I G
Y   I   R

-> "PAHNAPLSIIGYIR"
```

```
numRows = 4:
P     I    N
A   L S  I G
Y A   H R
P     I
-> "PINALSIGYAHRPI"
```

## How to explain it out loud

*"The most intuitive way to think about it is to actually build the grid. Track a current row and a direction — start at row 0 going down; every time you hit row 0 or the last row, flip direction. Walk through the string once, placing each character at (row, col), where col only advances while going in the 'up' direction (that's what actually creates the zigzag — going down stays in the same column, going up shifts one column right each step). Once the whole string is placed in the grid, just read it back row by row, skipping the empty cells."*

## Approach

Special case: if `numRows == 1`, there's no zigzag at all — return `s` unchanged (this also avoids a division-by-zero-flavored edge case in the row-bounce logic below).

Build a grid of size `numRows` × `n` (where `n = s.length()`), initialized to a placeholder blank character. Track `r` (current row), `c` (current column), and a direction flag `d`. Walk through `s` once: place `s[i]` at `matrix[r][c]`. If not currently going "up" (`d` false), just increment `r` (moving straight down a column). If going "up" (`d` true), increment `c` and decrement `r` (moving diagonally). Whenever `r` hits either boundary (`0` or `numRows-1`), flip `d`.

Once the grid is filled, read it back: for each row top to bottom, for each column left to right, append any non-blank character to the answer.

Time: O(n) to fill + O(numRows × n) to read (bounded by O(n) total, since only `n` cells are ever non-blank, but reading naively scans every cell) · Space: O(numRows × n) for the grid — more than strictly necessary, but simple to reason about and fine for LeetCode's constraints.

## Approach — Row strings, no grid (optimal)

Same row/direction bounce logic, but instead of a full `numRows × n` grid, keep just `numRows` separate strings — one per row. As you scan `s` once, append the current character directly to whichever row string is "current," then bounce the row index the same way (flip direction at row `0` or `numRows-1`). At the end, concatenate the row strings in order. No wasted space on blank cells, no separate read-out pass needed — the answer is just the rows joined together.

Time: O(n) · Space: O(n) total across all row strings (only real characters are ever stored, no padding)

## Solution

### C++
```cpp
class Solution {
public:
    string convert(string s, int numRows) {
        if (numRows == 1) return s;

        int n = s.length();
        vector<vector<char>> matrix(numRows, vector<char>(n, ' '));
        int r = 0, c = 0;
        bool d = false;  // false = moving down, true = moving diagonally up

        for (int i = 0; i < n; i++) {
            matrix[r][c] = s[i];
            if (!d) r++;
            else { c++; r--; }
            if (r == 0 || r == numRows-1) d = !d;
        }

        string ans;
        for (int r = 0; r < numRows; r++) {
            for (int c = 0; c < n; c++) {
                if (matrix[r][c] != ' ') {
                    ans += matrix[r][c];
                }
            }
        }
        return ans;
    }
};
```

### C++ (Row strings, no grid — optimal)
```cpp
class Solution {
public:
    string convert(string s, int numRows) {
        if (numRows == 1) return s;
        int n = s.length();
        vector<string> rows(min(numRows, n));
        int curRow = 0;
        bool goingDown = false;
        for (char ch : s) {
            rows[curRow] += ch;
            if (curRow == 0 || curRow == numRows - 1) goingDown = !goingDown;
            curRow += goingDown ? 1 : -1;
        }
        string ans;
        for (string& row : rows) ans += row;
        return ans;
    }
};
```

### Python
```python
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        if numRows == 1:
            return s
        n = len(s)
        matrix = [[' ']*n for _ in range(numRows)]
        r, c = 0, 0
        d = False
        for i in range(n):
            matrix[r][c] = s[i]
            if not d:
                r += 1
            else:
                c += 1
                r -= 1
            if r == 0 or r == numRows - 1:
                d = not d
        ans = []
        for r in range(numRows):
            for c in range(n):
                if matrix[r][c] != ' ':
                    ans.append(matrix[r][c])
        return "".join(ans)
```

### Java
```java
class Solution {
    public String convert(String s, int numRows) {
        if (numRows == 1) return s;

        int n = s.length();
        char[][] matrix = new char[numRows][n];
        for (char[] row : matrix) Arrays.fill(row, ' ');
        int r = 0, c = 0;
        boolean d = false;

        for (int i = 0; i < n; i++) {
            matrix[r][c] = s.charAt(i);
            if (!d) r++;
            else { c++; r--; }
            if (r == 0 || r == numRows-1) d = !d;
        }

        StringBuilder ans = new StringBuilder();
        for (int row = 0; row < numRows; row++) {
            for (int col = 0; col < n; col++) {
                if (matrix[row][col] != ' ') {
                    ans.append(matrix[row][col]);
                }
            }
        }
        return ans.toString();
    }
}
```

Verified against 10 cases (both problem examples, numRows=1, numRows greater than string length, and the important edge case numRows exactly equal to string length: `"AB"`/2, `"ABC"`/3, `"ABCD"`/4) — grid version and row-strings version cross-checked against each other and matched on all 10 (9 checked directly, the empty-string case handled by both trivially), C++ additionally clean under AddressSanitizer on both versions. Java is the grid version translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- Row/direction/grid-filling logic was correct on the first attempt — verified clean against both problem examples right away.
- Real bug in the reading loop: `if (matrix.size() == n) break;`, present in both the inner and outer read loops. `matrix.size()` is just `numRows` (fixed, doesn't depend on `r`/`c`/reading progress at all) and `n` is `s.length()` (also fixed) — so this condition secretly reduces to "is `numRows` exactly equal to the string's length," checked on every character appended, completely unrelated to what it looks like it's checking. Whenever that fixed condition happened to be true, both loops broke immediately after appending just the *first* character, silently truncating the output. Confirmed with concrete tests: `"AB"` with `numRows=2` returned `"A"` instead of `"AB"`; `"ABC"`/3 and `"ABCD"`/4 failed the same way. All other configurations (where `numRows != n`) passed by coincidence, which is why the bug wasn't obvious from the two standard problem examples. Fixed by removing both checks entirely — the two `for` loops already correctly cover every valid character in the grid without any early-exit condition needed.
