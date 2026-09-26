# Word Break (LeetCode Medium)

## Problem

Given a string `s` and a dictionary of strings `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words. The same word can be reused any number of times.

```
s = "leetcode", wordDict = ["leet","code"]        -> true   ("leet" + "code")
s = "applepenapple", wordDict = ["apple","pen"]    -> true   ("apple" + "pen" + "apple")
s = "catsandog", wordDict = ["cats","dog","sand","and","cat"] -> false
```

## How to explain it out loud

*"I define `dp[i]` as 'can the first `i` characters of `s` be fully split into dictionary words.' `dp[0]` is true trivially — an empty prefix is vacuously breakable. For each `i` from 1 up to the full length, I look backward for some split point `j` where `dp[j]` is already true and the chunk `s[j:i]` is itself a dictionary word — if I find one, `dp[i]` is true too, since I've just extended a valid split by one more word. To avoid checking every possible `j` back to 0, I only look back as far as the longest word in the dictionary, since no valid word can be longer than that. The final answer is just `dp[n]` — can the whole string be split."*

## Approach — Bottom-up DP with a max-word-length bound

Build a hash set of the dictionary words for O(1) membership checks, and track `maxLen`, the length of the longest word in `wordDict` (no valid split-ending chunk can ever be longer than this, so it bounds how far back to search).

`dp` is sized `n + 1` (not `n`) — `dp[i]` means "the first `i` characters of `s` can be split," so `dp[0]` (zero characters, the empty prefix) and `dp[n]` (the whole string) both need their own slot. `dp[0] = true`.

For `i` from `1` to `n`: search `j` from `i-1` down to `max(0, i - maxLen)`. At each `j`, check two things: is `dp[j]` already true (the prefix before this chunk is breakable), and is `s[j:i]` (the chunk from `j` up to but not including `i`, i.e. `s.substr(j, i-j)`) a dictionary word. If both hold, `dp[i] = true` and stop searching — one valid split is enough.

Answer: `dp[n]`.

Time: O(n × maxLen) — for each of `n` positions, the inner search looks back at most `maxLen` positions, each `substr` + hash lookup costing O(maxLen) for hashing/copying the chunk, so overall roughly O(n × maxLen²) in the strictest sense (dominated by substring construction), though commonly quoted as O(n × maxLen) treating the per-chunk hash/copy as bounded by the (small, fixed) max word length. Space: O(n) for the `dp` array plus O(total dictionary character count) for the word set.

## Solution

### C++ (bottom-up DP, max-length bound — your fixed version)
```cpp
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        //length max
        //have set and add all words
        unordered_set<string> words;
        int maxLen = 0;
        for(int i = 0; i<wordDict.size(); i++){
            maxLen = max(maxLen, (int)wordDict[i].length());
            words.insert(wordDict[i]);
        }
        //vector if the string can be splitted at the index
        vector<bool> splitPossible(s.length() + 1, false);
        splitPossible[0] = true;
        //for loop on each index
        for(int i=1; i<=s.length(); i++){
            //for loop to check if we can find matching word in set from length 1 to maxlen behind
            for(int j=i-1; j>=max(0,i-maxLen);j--){
                if(words.find(s.substr(j,i-j))!=words.end() && splitPossible[j]==true){
                    splitPossible[i] = true;
                    break;
                }
            }
        }
        return splitPossible[s.length()];
    }
};
```

### Python
```python
class Solution:
    def wordBreak(self, s: str, wordDict: list[str]) -> bool:
        words = set(wordDict)
        max_len = max(len(w) for w in wordDict) if wordDict else 0
        n = len(s)

        split_possible = [False] * (n + 1)
        split_possible[0] = True

        for i in range(1, n + 1):
            for j in range(i - 1, max(0, i - max_len) - 1, -1):
                if split_possible[j] and s[j:i] in words:
                    split_possible[i] = True
                    break

        return split_possible[n]
```

### Java
```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> words = new HashSet<>(wordDict);
        int maxLen = 0;
        for (String w : wordDict) maxLen = Math.max(maxLen, w.length());

        int n = s.length();
        boolean[] splitPossible = new boolean[n + 1];
        splitPossible[0] = true;

        for (int i = 1; i <= n; i++) {
            for (int j = i - 1; j >= Math.max(0, i - maxLen); j--) {
                if (splitPossible[j] && words.contains(s.substring(j, i))) {
                    splitPossible[i] = true;
                    break;
                }
            }
        }
        return splitPossible[n];
    }
}
```

Verified against 9 hand-picked cases (both classic true examples, the classic false example, a single unmatched character, a single matched character, an empty string, a repeated-word case requiring word reuse, an ambiguous-split case where a greedy longest-match would fail, and a case needing the dictionary's shortest word to complete a split) plus a 3000-trial randomized stress test (small alphabet, dictionary words up to length 4, strings up to length 14) against a memoized brute-force reference — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer (only harmless signed/unsigned comparison warnings on the loop bounds, not correctness bugs). Python and Java are the same DP translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- First attempt had three separate, real bugs, found via actual compilation and testing rather than inspection alone:
  - **Compile error**: `max(maxLen, wordDict[i].length())` mixed `int` with `size_t` (`.length()`'s return type) — `std::max` requires both arguments to be the same type, so this failed to compile at all. Fixed by casting: `max(maxLen, (int)wordDict[i].length())`.
  - **Wrong substring**: `s.substr(j, i)` — `substr`'s second argument is a *length*, not an end index, so this extracted the wrong characters entirely (a chunk of length `i` starting at `j`, instead of the intended chunk from `j` up to `i`). Fixed to `s.substr(j, i - j)`.
  - **Out-of-bounds return**: `splitPossible` was sized `s.length()` (only indices `0..n-1`) with no slot representing "the whole string is breakable," and the return statement was `splitPossible[-1]` — C++ doesn't support Python-style negative indexing; `vector::operator[]` takes an unsigned index, so `-1` silently wrapped to a huge value and read out-of-bounds memory, confirmed as a real crash under AddressSanitizer (heap-buffer-overflow). Fixed by resizing `splitPossible` to `s.length() + 1` (adding a slot for the empty-prefix base case and the full-string answer) and returning `splitPossible[s.length()]`, with the loop range adjusted from `i` in `[0, n)` to `i` in `[1, n]` and the substring/base-case logic updated to match the new indexing convention (`splitPossible[i]` now means "the first `i` characters are breakable," not "index `i` is breakable").
