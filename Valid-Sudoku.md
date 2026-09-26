# Valid Sudoku (LeetCode Medium)

## Problem

Given a 9x9 Sudoku board, partially filled with `'1'-'9'` and `'.'` for empty cells, determine if the *currently filled* cells satisfy Sudoku's rules — no repeated digit in any row, any column, or any of the nine 3x3 boxes. The board doesn't need to be solvable or fully filled, just consistent so far.

## How to explain it out loud

*"I do a single pass over every cell. For each filled cell, I need to check three things at once: has this digit already appeared in this row, this column, or this 3x3 box? I track that with three 9x9 tables — `row[i][digit]`, `col[j][digit]`, `box[boxIndex][digit]` — each a boolean I flip on the first time I see that digit in that row/column/box. If any of the three is already set when I reach a cell, that's a violation, return false immediately. The only fiddly part is mapping a cell's `(i,j)` to which of the 9 boxes it belongs to: dividing both `i` and `j` by 3 gives which 'super-row' and 'super-column' of boxes it's in, and combining those two into a single 0-8 index with `3*(i/3) + (j/3)` gives a unique, consistent box id."*

## Approach — Single pass, three seen-tables (row/column/box)

Three `9x9` int (or bool) grids: `row[i][digit]`, `col[j][digit]`, `box[b][digit]`, all initialized to 0/false. Walk every cell `(i, j)`. Skip `'.'`. For a filled cell, convert the character to a 0-8 index with `digit = board[i][j] - '1'`. Check and set `row[i][digit]`; if it was already set, return `false`. Same for `col[j][digit]`. For the box, compute `boxInd = 3*(i/3) + (j/3)` — integer division groups rows 0-2/3-5/6-8 and columns 0-2/3-5/6-8 into 3 groups each, and the combination `3*rowGroup + colGroup` gives a unique index 0-8 for each of the nine 3x3 regions. Check and set `box[boxInd][digit]`, same as the others.

If every cell passes all three checks, return `true`.

Time: O(1) — the board is always exactly 9x9 by problem definition, so the "81 cells, 3 O(1) checks each" work is a fixed constant, not a function of a growing input; still commonly described as O(n²) for a general n×n grid if this pattern were reused elsewhere. Space: O(1) — the three tracking tables are fixed-size (9x9 each) regardless of board content.

## Solution

### C++
```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        int n = board[0].size();
        vector<vector<int>> row (9,vector<int>(9, 0));
        vector<vector<int>> col (9,vector<int>(9, 0));
        vector<vector<int>> box (9,vector<int>(9, 0));

        for(int i=0; i<n; i++){
            for(int j=0; j<n; j++){
                if(board[i][j]=='.') continue;

                int digit = board[i][j] - '1';
                if(row[i][digit]) return false;
                row[i][digit] = 1;

                if(col[j][digit]) return false;
                col[j][digit] = 1;

                int boxSize = n/3;
                int boxInd = boxSize * (i/boxSize) + (j/boxSize);
                if(box[boxInd][digit]) return false;
                box[boxInd][digit] = 1;
            }
        }
        return true;
    }
};
```

### Python
```python
class Solution:
    def isValidSudoku(self, board: list[list[str]]) -> bool:
        row = [[False] * 9 for _ in range(9)]
        col = [[False] * 9 for _ in range(9)]
        box = [[False] * 9 for _ in range(9)]

        for i in range(9):
            for j in range(9):
                if board[i][j] == '.':
                    continue
                digit = ord(board[i][j]) - ord('1')

                if row[i][digit]:
                    return False
                row[i][digit] = True

                if col[j][digit]:
                    return False
                col[j][digit] = True

                box_ind = 3 * (i // 3) + (j // 3)
                if box[box_ind][digit]:
                    return False
                box[box_ind][digit] = True

        return True
```

### Java
```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        boolean[][] row = new boolean[9][9];
        boolean[][] col = new boolean[9][9];
        boolean[][] box = new boolean[9][9];

        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] == '.') continue;
                int digit = board[i][j] - '1';

                if (row[i][digit]) return false;
                row[i][digit] = true;

                if (col[j][digit]) return false;
                col[j][digit] = true;

                int boxInd = 3 * (i / 3) + (j / 3);
                if (box[boxInd][digit]) return false;
                box[boxInd][digit] = true;
            }
        }
        return true;
    }
}
```

Verified against 7 cases: LeetCode's own official valid-board example, LeetCode's own invalid example (a duplicate `8` sharing both a column and a box), an entirely empty board (all `'.'`), a same-row duplicate, a same-column duplicate, a same-box duplicate that shares neither row nor column (confirming the box check specifically, not just piggybacking on row/col), and two `'9'`s (digit index 8 — the tricky upper array bound) placed in different rows/columns/boxes to confirm no false positive at the boundary — all correct, clean under AddressSanitizer + UndefinedBehaviorSanitizer.

## Bonus — generalizing to any board size `n` that's a multiple of 3

A natural follow-up: does the same approach generalize to an `n x n` board (box height/width still fixed at 3, digits `'1'` through the character for `n`)? A first attempt just replaced every hardcoded `9` with `n`:

```cpp
// buggy "generalized" attempt
vector<vector<int>> box (n, vector<int>(n, 0));
...
int boxSize = n/3;
int boxInd = boxSize * (i/boxSize) + (j/boxSize);
```

This is wrong, and only *looks* right because it happens to reduce to the correct `n=9` code — it does not actually generalize. Two independent bugs:

- **Box array sized by `n` instead of a fixed `9`.** With box height/width fixed at 3, there are always exactly 9 boxes (a 3×3 grid of them) no matter how big the board is — the box *count* doesn't scale with `n`, only the box *contents* (digit range) does. Sizing the outer `box` vector to `n` rows instead of a fixed `9` breaks for any `n < 9`: confirmed a real out-of-bounds crash under AddressSanitizer for `n=3` (a filled bottom-right cell computes `boxInd = 4`, but `box` only has 3 rows).
- **Multiplier in `boxInd` should be fixed at `3`, not `boxSize`.** The formula `multiplier * (i/boxSize) + (j/boxSize)` only produces a collision-free 0-8 mapping when `multiplier` equals the number of box-*columns*, which is always `3` (fixed box width) — not `boxSize` (`n/3`, the box's own side length). These two quantities only coincide when `boxSize == 3`, i.e. exactly `n == 9`. Confirmed a real silent bug for `n=6` (`boxSize=2`): two `'5'`s placed in genuinely different, non-overlapping boxes ((2,4) and (4,0)) both hashed to `boxInd=4` and were incorrectly flagged as a duplicate — a false positive, not a crash, which is the more dangerous kind of bug since it doesn't announce itself.

Fixed version — box array fixed at 9 rows, multiplier for `boxInd` fixed at 3:

```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        int n = board[0].size();
        vector<vector<int>> row (n,vector<int>(n, 0));
        vector<vector<int>> col (n,vector<int>(n, 0));
        int boxSize = n/3;
        vector<vector<int>> box (9, vector<int>(n, 0));  // always 9 boxes, regardless of n

        for(int i=0; i<n; i++){
            for(int j=0; j<n; j++){
                if(board[i][j]=='.') continue;

                int digit = board[i][j] - '1';
                if(row[i][digit]) return false;
                row[i][digit] = 1;

                if(col[j][digit]) return false;
                col[j][digit] = 1;

                int boxInd = 3 * (i/boxSize) + (j/boxSize);  // multiplier fixed at 3 (box-columns), not boxSize
                if(box[boxInd][digit]) return false;
                box[boxInd][digit] = 1;
            }
        }
        return true;
    }
};
```

Verified: the n=9 official example still passes; the n=6 false-collision case now correctly returns valid; a genuine n=6 same-box duplicate is still correctly caught; the n=3 crash is gone and the bottom-right-filled case now correctly returns valid; plus a 900-trial randomized stress test across n=3, 6, 9 against an independent set-based per-row/column/box reference — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer.

## Bug log

- Original n=9-only version: correct on the first attempt — no bugs found.
- "Generalized to any n" attempt: two real bugs, both requiring actual execution (not just inspection) to surface — an out-of-bounds crash for `n=3` from sizing the box array by `n` instead of the true fixed box count of 9, and a silent false-positive duplicate for `n=6` from using `boxSize` instead of the fixed value `3` as the `boxInd` multiplier. Both fixed as shown above; the fix coincidentally does nothing different for `n=9`, which is exactly why the bug was invisible until tested against other values of `n`.
