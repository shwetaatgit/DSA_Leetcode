# Isomorphic Strings (LeetCode Easy)

## Problem

Given two strings `s` and `t`, determine if they are isomorphic — all occurrences of a character in `s` can be replaced to get `t`, with two conditions: no two characters in `s` may map to the same character in `t`, and the mapping must be consistent (every occurrence of a character always maps to the same target).

```
s = "egg", t = "add"     -> true
s = "foo", t = "bar"     -> false
s = "paper", t = "title" -> true
s = "ab", t = "aa"       -> false   (two different s-characters both map to 'a')
```

## How to explain it out loud

*"The mapping has to work both ways — every character in s always maps to the same character in t, AND no two different characters in s are allowed to map to the same character in t. So I need two separate hashmaps, not one: s-to-t and t-to-s. Walk through both strings together. If the current s-character has been seen before, its existing mapping must match the current t-character, otherwise it's not isomorphic. Independently, if the current t-character has been seen before, its existing mapping must match the current s-character. Only if a character is brand new in both directions do I record a fresh pairing. Checking both directions independently, with independent maps, is what actually enforces the bijection — using just one map, or one shared map for both directions, misses cases where two different s-characters collapse onto the same t-character."*

## Approach

Two separate hashmaps: `s2t` (maps a seen `s` character to the `t` character it must always align with) and `t2s` (the reverse). Walk through both strings position by position. At each position, check `s2t` for `s[i]`: if it already has a mapping and that mapping isn't `t[i]`, fail. Check `t2s` for `t[i]`: if it already has a mapping and that mapping isn't `s[i]`, fail. Otherwise, record both directions: `s2t[s[i]] = t[i]` and `t2s[t[i]] = s[i]`.

Both directions must be checked *independently*, with independent storage — one map can only naturally enforce one direction of the constraint (that a given `s` character always produces the same `t` character); it says nothing about whether some *other* `s` character has already claimed that same `t` character.

Time: O(n) · Space: O(1) effectively (bounded by alphabet size)

## Solution

### C++
```cpp
class Solution {
public:
    bool isIsomorphic(string s, string t) {
        if (s.length() != t.length()) return false;
        map<char,char> s2t, t2s;
        for (int i = 0; i < (int)s.length(); i++) {
            if (s2t.find(s[i]) == s2t.end() && t2s.find(t[i]) == t2s.end()) {
                s2t[s[i]] = t[i];
                t2s[t[i]] = s[i];
            } else {
                if (s2t.find(s[i]) == s2t.end() || s2t[s[i]] != t[i]) return false;
                if (t2s.find(t[i]) == t2s.end() || t2s[t[i]] != s[i]) return false;
            }
        }
        return true;
    }
};
```

### Python
```python
class Solution:
    def isIsomorphic(self, s: str, t: str) -> bool:
        if len(s) != len(t):
            return False
        s2t, t2s = {}, {}
        for cs, ct in zip(s, t):
            if cs in s2t and s2t[cs] != ct:
                return False
            if ct in t2s and t2s[ct] != cs:
                return False
            s2t[cs] = ct
            t2s[ct] = cs
        return True
```

### Java
```java
class Solution {
    public boolean isIsomorphic(String s, String t) {
        if (s.length() != t.length()) return false;
        Map<Character, Character> s2t = new HashMap<>();
        Map<Character, Character> t2s = new HashMap<>();
        for (int i = 0; i < s.length(); i++) {
            char cs = s.charAt(i), ct = t.charAt(i);
            if (s2t.containsKey(cs) && s2t.get(cs) != ct) return false;
            if (t2s.containsKey(ct) && t2s.get(ct) != cs) return false;
            s2t.put(cs, ct);
            t2s.put(ct, cs);
        }
        return true;
    }
}
```

Verified against 7 hand-picked cases (both classic true/false examples, a longer true example, a two-characters-collapse-to-one case, a similar collapse embedded mid-string, and two cases confirming a valid bijection isn't wrongly rejected) plus a 5000-trial randomized stress test (short strings over a 4-letter alphabet, to force frequent character collisions) against an independent reference implementation — 0 mismatches. C++ clean under AddressSanitizer. Python matches on all 7 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- First attempt used a single `map<char,char> m`, checking only `s[i]` as a key: if unseen, insert `m[s[i]]=t[i]`; if seen, verify `t[i]==m[s[i]]`. This only enforces the forward direction (a given `s` character always maps to the same `t` character) — it never checks whether a *different* `s` character has already claimed that same `t` character. Confirmed as a real bug: `s="ab"`, `t="aa"` returned `true` (should be `false` — both `'a'` and `'b'` map to `'a'`), and `s="badc"`, `t="baba"` similarly returned `true` incorrectly.
- Second attempt tried to fix this by writing to the *same* map in both directions — `m[s[i]]=t[i]` and `m[t[i]]=s[i]` — still gated by a single existence check on `s[i]` alone. This introduced a **new** bug on top of not fixing the original one: since only `s[i]`'s existence was checked before both writes fired, the second write (`m[t[i]]=s[i]`) could silently overwrite an unrelated, previously-valid mapping whenever `t[i]` happened to already be a key from some earlier position — with no conflict check at all for that overwrite. Confirmed with `s="aba"`, `t="cac"` (a genuinely valid isomorphism) wrongly returning `false`: at `i=0`, `m['a']='c'` gets set; at `i=1` (`s='b'`, `t='a'`), since `'b'` is new, the insert branch overwrites `m['a']` to `'b'`, destroying the valid `'a'→'c'` mapping before `i=2` ever checks it. Meanwhile the *original* missing-case bug (`"ab"`/`"aa"`) was still present too, for the same underlying reason: the shared single existence-check on `s[i]` never independently validates `t[i]`'s prior claims. A 2000-trial stress test at this stage showed 212 mismatches.
- Fixed by using two fully independent maps (`s2t`, `t2s`), each with its own existence check and its own write, so neither direction's bookkeeping can ever silently corrupt the other's. Verified afterward with 0 mismatches across 5000 random trials.
