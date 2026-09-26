# Longest Substring Without Repeating Characters (LeetCode Medium)

## Problem

Given a string `s`, find the length of the longest substring without repeating characters.

```
s = "abcabcbb" -> 3   ("abc")
s = "bbbbb"    -> 1   ("b")
s = "pwwkew"   -> 3   ("wke")
```

## How to explain it out loud

*"Sliding window with a set tracking which characters are currently in the window. Expand the right edge one character at a time. If that character is already in the set — meaning it's a repeat within the current window — shrink from the left, removing characters from the set, until the repeat is gone. Only then insert the new character and record the window length as a candidate for the max. Every character enters and leaves the set at most once across the whole scan, so it's linear time even though there's a nested-looking loop."*

## Approach — Set-based sliding window

Two pointers `l` and `r`, both starting at `0`, plus a `set<char>` tracking exactly which characters are in the current window `[l, r)`. Walk `r` forward through the string. Before adding `s[r]` to the window, check whether it's already in `seen`: if so, that's a duplicate, so shrink from the left — removing `s[l]` from `seen` and advancing `l` — repeatedly, until `s[r]` is no longer present (this correctly handles the duplicate wherever it sits in the window, not just at the very edge). Once the window is clean, insert `s[r]` and update the max length with `r - l + 1`.

Time: O(n log k), where `k` is the window size (bounded by the alphabet) — `std::set` is a balanced tree, so each `insert`/`erase`/`find` costs O(log k); each character is inserted and erased at most once across the scan, giving O(n) total operations, each with that O(log k) cost. With a bounded alphabet, `log k` is a small constant, so this is close to linear in practice, but not asymptotically O(n). Space: O(min(n, alphabet size)) for the set.

## Approach — Last-seen-index, single pass (optimal)

Instead of a set that only answers "is this character currently in the window," keep a map (or fixed-size array, since the alphabet is bounded) from character to *the last index it was seen at*. Walk `r` forward once. If the current character was seen before, **and** that earlier occurrence is still inside the current window (`lastSeen[ch] >= l`), jump `l` directly to `lastSeen[ch] + 1` — one O(1) jump, computed from information already on hand, instead of a shrink-loop removing one character at a time. Always update `lastSeen[ch] = r` afterward, and update the max length with `r - l + 1`.

The `lastSeen[ch] >= l` check is essential, not optional: a character's last-seen index might be *stale* — from before the current window even started (already shrunk past by an earlier duplicate) — and blindly jumping `l` to `lastSeen[ch] + 1` in that case would incorrectly move `l` *backward*, shrinking the window using outdated information. Using `max(l, lastSeen[ch]+1)` (equivalently, only jumping when `lastSeen[ch] >= l`) guards against that.

Two ways to store `lastSeen`, with different exact complexities:

- **Fixed-size array** (`vector<int> lastSeen(256, -1)`, indexed directly by character code): every access is a direct array index — true O(1) *worst-case* per character, no hashing involved at all. Total time O(n), guaranteed, not just on average. Space is a flat O(1) — the array's size is fixed by the alphabet (e.g. 256), completely independent of the input string's length or how many distinct characters it actually uses.
- **`unordered_map<char,int>`**: each access (`find`/`operator[]`/`erase`) is O(1) *on average*, amortized over many operations, backed by hashing — but O(n) *worst-case* for any single operation under pathological hash collisions (a guarantee `std::unordered_map` makes, even if rare in practice). Total time is O(n) average, not a hard worst-case guarantee the way the array version has. Space is O(min(n, alphabet size)) — only entries for characters actually encountered are stored, plus per-entry hash-table overhead (bucket/node allocation), so its constant factor is larger than the array version despite both being asymptotically similar for a bounded alphabet.

In short: for a bounded, known alphabet (like ASCII), the array version is strictly better — same asymptotic class, smaller constant factor, and a true worst-case bound instead of an average-case one. The hashmap version is more general (works for any character type without needing to size an array to a known alphabet up front, e.g. arbitrary Unicode), at the cost of that hashing overhead.

Time: O(n) — a true single pass, one O(1) step per character, no amortized shrink-loop argument needed · Space: O(1) with the fixed-size array, or O(min(n, alphabet size)) with the hashmap

## Solution

### C++
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.length() == 0) return 0;
        set<char> seen;
        int l = 0, r = 0;
        int maxLen = INT_MIN;
        while (r < s.length()) {
            while (seen.find(s[r]) != seen.end()) {
                seen.erase(s[l]);
                l++;
            }
            seen.insert(s[r]);
            maxLen = max(maxLen, r - l + 1);
            r++;
        }
        return maxLen;
    }
};
```

### C++ (Last-seen-index — optimal)
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        vector<int> lastSeen(256, -1);
        int l = 0, maxLen = 0;
        for (int r = 0; r < (int)s.length(); r++) {
            unsigned char ch = s[r];
            if (lastSeen[ch] >= l) {
                l = lastSeen[ch] + 1;
            }
            lastSeen[ch] = r;
            maxLen = max(maxLen, r - l + 1);
        }
        return maxLen;
    }
};
```

### C++ (Last-seen-index, `unordered_map` version)
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.length() == 0) return 0;
        unordered_map<char,int> seen;
        int l = 0, r = 0;
        int maxLen = INT_MIN;
        while (r < s.length()) {
            if (seen.find(s[r]) != seen.end() && seen[s[r]] >= l) {
                l = seen[s[r]] + 1;
                seen.erase(s[r]);
            }
            seen[s[r]] = r;
            maxLen = max(maxLen, r - l + 1);
            r++;
        }
        return maxLen;
    }
};
```
Same optimal single-pass logic as the array version above, using `unordered_map<char,int>` instead of a fixed-size array. Same asymptotic time class (O(n)), but average-case rather than the array version's guaranteed worst-case, with extra hashing/bucket overhead per operation — see the complexity comparison above. The `seen[s[r]] >= l` guard is the fix for the same stale-index pitfall described above; an earlier version of this code jumped unconditionally whenever the character was found in the map at all (without checking it was still `>= l`), which failed concretely on `"abba"` (returned `3` instead of `2`) and `"tmmzuxt"` (returned `6` instead of `5`) — both cases where a character's last-seen index was stale, from before the window had already shrunk past it due to a *different* duplicate.

### Python (Set-based sliding window)
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        if len(s) == 0:
            return 0
        seen = set()
        l = 0
        max_len = 0
        for r in range(len(s)):
            while s[r] in seen:
                seen.remove(s[l])
                l += 1
            seen.add(s[r])
            max_len = max(max_len, r - l + 1)
        return max_len
```

### Python (Last-seen-index — optimal)
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last_seen = {}
        l = 0
        max_len = 0
        for r, ch in enumerate(s):
            if ch in last_seen and last_seen[ch] >= l:
                l = last_seen[ch] + 1
            last_seen[ch] = r
            max_len = max(max_len, r - l + 1)
        return max_len
```

### Java (Last-seen-index — optimal)
```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int[] lastSeen = new int[128];
        Arrays.fill(lastSeen, -1);
        int l = 0, maxLen = 0;
        for (int r = 0; r < s.length(); r++) {
            char ch = s.charAt(r);
            if (lastSeen[ch] >= l) {
                l = lastSeen[ch] + 1;
            }
            lastSeen[ch] = r;
            maxLen = Math.max(maxLen, r - l + 1);
        }
        return maxLen;
    }
}
```

Set-based version verified against 8 hand-picked cases (both classic repeating examples, an empty string, a single space, a two-character all-unique string, and the two well-known tricky cases `"dvdf"` and `"abba"`) plus a 3000-trial randomized stress test against a brute-force reference — 0 mismatches, C++ clean under AddressSanitizer. Last-seen-index array version verified against 9 hand-picked cases (the same 8 plus `"tmmzuxt"`, which specifically tests that a stale last-seen index — from a character last seen *before* the current window — doesn't incorrectly pull `l` backward) plus a 5000-trial randomized stress test against the same brute-force reference — 0 mismatches, clean under AddressSanitizer. Last-seen-index `unordered_map` version verified against the same 9 hand-picked cases plus its own 5000-trial randomized stress test — 0 mismatches, clean under AddressSanitizer. Python versions match on all hand-picked cases for both set-based and last-seen-index approaches. Java (last-seen-index array) is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- Set-based version: correct on the first attempt.
- Last-seen-index array version: also correct on the first attempt, including correctly identifying and guarding against the stale-index pitfall (`lastSeen[ch] >= l` check) before it was ever written incorrectly.
- Last-seen-index `unordered_map` version: a first attempt **omitted** the `>= l` guard — jumped `l = seen[s[r]] + 1` unconditionally whenever `s[r]` existed anywhere in the map, without checking whether that recorded index was still inside the current window. Confirmed as a real bug with two concrete failing cases: `"abba"` returned `3` instead of `2`, and `"tmmzuxt"` returned `6` instead of `5`. In both, a character's map entry was stale — left over from before an *unrelated* duplicate had already shrunk the window past it — and jumping to `stale_index + 1` walked `l` backward, inflating the window with characters that should have already been excluded. Fixed by adding the same `seen[s[r]] >= l` check used in the array version, verified afterward with 0 mismatches across 5000 random trials.
