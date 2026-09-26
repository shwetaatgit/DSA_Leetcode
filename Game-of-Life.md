# Game of Life (LeetCode Medium)

## Problem

Given an `m x n` board where each cell is `1` (live) or `0` (dead), update the board in place to its next state, per Conway's rules: a live cell with 2 or 3 live neighbors survives (otherwise dies); a dead cell with exactly 3 live neighbors becomes alive. Neighbors are the up-to-8 surrounding cells. All cells update simultaneously, based on the *current* board, not a partially-updated one.

## How to explain it out loud

*"The tricky part of doing this in place is that a naive in-place update would corrupt the board partway through — by the time I get to a later cell, some of its neighbors might already reflect the *next* generation instead of the current one, throwing off its neighbor count. The trick is to encode 'this cell changed' using two extra states instead of overwriting immediately: if a live cell is about to die, I mark it `2` instead of `0`; if a dead cell is about to come alive, I mark it `3` instead of `1`. The key property that makes this safe is that both `1` and `2` still mean 'this cell was originally alive' — so when I'm counting a neighbor's live/dead status for some *other* cell I haven't processed yet, checking `value == 1 || value == 2` always tells me the neighbor's *original* state, regardless of whether I've already visited and modified that neighbor in this same pass. Once the whole board's been scanned and marked with 2s and 3s, a second, simple pass converts `2 → 0` and `3 → 1`, and the encoding's job is done."*

## Approach 1 — Brute force, extra neighbor-count array

The straightforward version, before reaching for any in-place trick: allocate a second `rows x cols` array, `neighborCount`, and fill it in one pass by counting live neighbors of every cell — reading only from the original `board`, which is never touched during this pass, so there's no ordering concern at all (every count is computed from the one, true, unmodified starting state).

Only *after* every cell's count has been fully computed does a second pass overwrite `board` itself: for each cell, apply the rules using `board[i][j]`'s original value (still intact, since pass 1 never wrote to `board`) together with the now-fully-computed `neighborCount[i][j]`.

Time: O(rows × cols) — same two passes, same O(8) neighbor work per cell in the first pass · Space: O(rows × cols) for the extra `neighborCount` array — this is the cost of avoiding the in-place encoding trick; simpler to reason about, but not in-place.

## Approach 2 — In-place, two extra sentinel states (optimal)

For each cell `(i,j)`, count live neighbors among the 8 surrounding cells, checking `board[ni][nj] == 1 || board[ni][nj] == 2` — this correctly identifies "was originally alive" whether or not that neighbor has already been updated in this pass, since original-live cells only ever hold `1` (unchanged) or `2` (marked dying) at any point during the scan, never `0` or `3`.

Apply the rules to decide the new state:
- Currently alive (`board[i][j] == 1`): if the live-neighbor count is outside `{2, 3}`, mark it `2` (alive now, dead next). Otherwise leave it `1`.
- Currently dead (`board[i][j] == 0`): if the live-neighbor count is exactly `3`, mark it `3` (dead now, alive next). Otherwise leave it `0`.

After the full first pass, every cell holds one of `{0, 1, 2, 3}`, where `0`/`1` mean "unchanged" and `2`/`3` mean "changed." A second pass converts `2 → 0` and `3 → 1`, finalizing the board.

Time: O(rows × cols) — two passes over the board, each cell doing O(8) neighbor work in the first pass and O(1) in the second · Space: O(1) extra — no auxiliary board copy needed, unlike the more obvious approach of building an entirely new board and swapping it in.

## Solution

### C++ (brute force — extra neighbor-count array)
```cpp
class Solution {
public:
    void gameOfLife(vector<vector<int>>& board) {
        int rows = board.size();
        if (rows == 0) return;
        int cols = board[0].size();

        // extra vector: live-neighbor count for every cell, computed from the
        // original (unmodified) board -- no ordering concerns at all, since
        // board itself isn't touched during this pass
        vector<vector<int>> neighborCount(rows, vector<int>(cols, 0));

        int dr[] = {-1,-1,-1,0,0,1,1,1};
        int dc[] = {-1,0,1,-1,1,-1,0,1};

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                int count = 0;
                for (int d = 0; d < 8; d++) {
                    int ni = i + dr[d], nj = j + dc[d];
                    if (ni >= 0 && ni < rows && nj >= 0 && nj < cols && board[ni][nj] == 1) {
                        count++;
                    }
                }
                neighborCount[i][j] = count;
            }
        }

        // second pass: now safe to overwrite board directly, since every
        // neighborCount[i][j] was already fully computed from the untouched original
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (board[i][j] == 1) {
                    if (neighborCount[i][j] < 2 || neighborCount[i][j] > 3) {
                        board[i][j] = 0;
                    }
                } else {
                    if (neighborCount[i][j] == 3) {
                        board[i][j] = 1;
                    }
                }
            }
        }
    }
};
```

### C++ (optimal — in-place, sentinel states)
```cpp
class Solution {
public:
    void gameOfLife(vector<vector<int>>& board) {
        int rows = board.size();
        if (rows == 0) return;
        int cols = board[0].size();

        int dr[] = {-1,-1,-1,0,0,1,1,1};
        int dc[] = {-1,0,1,-1,1,-1,0,1};

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                int liveNeighbors = 0;
                for (int d = 0; d < 8; d++) {
                    int ni = i + dr[d], nj = j + dc[d];
                    if (ni >= 0 && ni < rows && nj >= 0 && nj < cols) {
                        if (board[ni][nj] == 1 || board[ni][nj] == 2) {
                            liveNeighbors++;
                        }
                    }
                }

                if (board[i][j] == 1) {
                    if (liveNeighbors < 2 || liveNeighbors > 3) {
                        board[i][j] = 2; // alive -> dead
                    }
                } else {
                    if (liveNeighbors == 3) {
                        board[i][j] = 3; // dead -> alive
                    }
                }
            }
        }

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (board[i][j] == 2) board[i][j] = 0;
                else if (board[i][j] == 3) board[i][j] = 1;
            }
        }
    }
};
```

### Python
```python
class Solution:
    def gameOfLife(self, board: list[list[int]]) -> None:
        rows = len(board)
        if rows == 0:
            return
        cols = len(board[0])
        directions = [(-1,-1),(-1,0),(-1,1),(0,-1),(0,1),(1,-1),(1,0),(1,1)]

        for i in range(rows):
            for j in range(cols):
                live_neighbors = 0
                for di, dj in directions:
                    ni, nj = i + di, j + dj
                    if 0 <= ni < rows and 0 <= nj < cols and board[ni][nj] in (1, 2):
                        live_neighbors += 1

                if board[i][j] == 1:
                    if live_neighbors < 2 or live_neighbors > 3:
                        board[i][j] = 2  # alive -> dead
                else:
                    if live_neighbors == 3:
                        board[i][j] = 3  # dead -> alive

        for i in range(rows):
            for j in range(cols):
                if board[i][j] == 2:
                    board[i][j] = 0
                elif board[i][j] == 3:
                    board[i][j] = 1
```

### Java
```java
class Solution {
    public void gameOfLife(int[][] board) {
        int rows = board.length;
        if (rows == 0) return;
        int cols = board[0].length;
        int[] dr = {-1,-1,-1,0,0,1,1,1};
        int[] dc = {-1,0,1,-1,1,-1,0,1};

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                int liveNeighbors = 0;
                for (int d = 0; d < 8; d++) {
                    int ni = i + dr[d], nj = j + dc[d];
                    if (ni >= 0 && ni < rows && nj >= 0 && nj < cols) {
                        if (board[ni][nj] == 1 || board[ni][nj] == 2) liveNeighbors++;
                    }
                }
                if (board[i][j] == 1) {
                    if (liveNeighbors < 2 || liveNeighbors > 3) board[i][j] = 2;
                } else {
                    if (liveNeighbors == 3) board[i][j] = 3;
                }
            }
        }

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (board[i][j] == 2) board[i][j] = 0;
                else if (board[i][j] == 3) board[i][j] = 1;
            }
        }
    }
}
```

Verified both the brute-force and in-place versions against 6 hand-picked boards (the classic LeetCode glider-ish example, a full 2x2 block, a single dead cell, a single live cell, an all-dead board, and an all-alive 3x3 board) plus, for each version, its own 1000-trial randomized stress test (random dimensions 1-8, random 0/1 fill) against an independent copy-based reference (computes the next generation into a fresh board without any in-place tricks) — 0 mismatches for both, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same in-place (optimal) encoding translated directly (Java not independently compiled in this environment — no JDK available).

*(Side note purely for interest, not part of the submitted solution: `board[i][j] % 2` actually gives the final next-state value directly for all four codes — `0→0`, `1→1`, `2→0`, `3→1` — so the cleanup pass could alternatively be replaced by taking every cell mod 2. Not used above since the explicit `if/else if` is clearer to read and reason about in an interview.)*

## Bug log

- Correct on the first attempt — no bugs found. The core insight (encoding "changed" as `+2` while keeping the original state recoverable via `value == 1 || value == 2`) was verified as sound before writing any code, by checking that no code path ever produces a live cell holding `3` or a dead cell holding `2`.
