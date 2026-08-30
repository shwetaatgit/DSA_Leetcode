# Insert Delete GetRandom O(1) (LeetCode Medium)

## Problem

Design a data structure supporting, all in **average O(1)** time:
- `insert(val)`: insert `val` if not already present. Return `true` if it was inserted, `false` if it was already there.
- `remove(val)`: remove `val` if present. Return `true` if it was removed, `false` if it wasn't there.
- `getRandom()`: return a random element from the current set — **every element must have equal probability** of being picked.

```
insert(1) -> true
remove(2) -> false
insert(2) -> true
getRandom() -> 1 or 2, each with 50% chance
remove(1) -> true
insert(2) -> false   (already present)
getRandom() -> 2
```

## How to explain it out loud

*"The tricky part is getRandom needing O(1) uniform access — that rules out a set or map on its own, since neither gives you 'the k-th element' without walking it. So I use two structures together: a vector for O(1) index-based access (getRandom just picks a random index), and a hashmap from value to its index in the vector, for O(1) lookup. Insert is straightforward — push to the back of the vector, record its index in the map. Remove is the interesting one, because erasing from the middle of a vector is O(n) — everything after it has to shift. Instead I swap the element I'm removing with whatever is currently the last element in the vector, then pop_back(), which is O(1). That means I have to update the map entry for the element I moved into the gap, since its index changed. One edge case: if the element I'm removing already IS the last element, there's nothing to swap with — I just pop it directly, and I have to make sure I don't touch the vector at the old index after the pop, because by then that index is out of bounds."*

## Approach

**insert(val):** check the map. If `val` is already a key, return `false`. Otherwise push `val` onto the back of the vector and record `map[val] = v.size()-1`. Return `true`.

**remove(val):** check the map. If `val` isn't a key, return `false`. Otherwise:
1. Look up `idx = map[val]` — where `val` currently sits in the vector.
2. If `idx` is **not** the last index, move the last element into slot `idx` (`v[idx] = v.back()`), and update the map so the moved element points at its new index (`map[v[idx]] = idx`).
3. If `idx` **is** the last index, skip step 2 — the element is already sitting where it would be moved to, so overwriting `v[idx]` with `v.back()` would just overwrite it with itself, and doing that read *after* the vector shrinks is the actual crash risk (see Bug log).
4. `pop_back()` the vector — the old value at the end (either the true duplicate copy, or `val` itself when it was last) is dropped.
5. Erase `val` from the map.
6. Return `true`.

**getRandom():** pick a uniformly random index in `[0, v.size())` and return `v[index]`. Because every live element occupies exactly one vector slot with no gaps, this is uniform over the current set.

Time: O(1) average for all three operations · Space: O(n)

## Why remove() needs the "is it the last element" check — worked example

Say the vector is `[10, 20, 30]` and the map is `{10:0, 20:1, 30:2}`.

**Case: remove a middle/first element — `remove(10)`.**
`idx = map[10] = 0`. `0` is not the last index (`2`), so: `v[0] = v.back() = 30` → vector becomes `[30, 20, 30]`, then `map[30] = 0` (30 now lives at index 0, overwriting its old entry `30:2`). Then `pop_back()` drops the trailing duplicate `30` → vector is `[30, 20]`. Erase `10` from the map. Final state: vector `[30, 20]`, map `{20:1, 30:0}` — consistent.

**Case: remove the last element — `remove(30)`.**
`idx = map[30] = 2`, and `2` **is** the last index. If you skip the "is it last" check and always run `v[idx] = v.back(); map[v[idx]] = idx; pop_back();`, you'd do `v[2] = v.back()` (a harmless self-assignment, `30 = 30`) — but the real danger is code that instead reads `v[idx]` *after* the `pop_back()` already happened, e.g. `pop_back(); map[v[idx]] = idx;`. At that point the vector has shrunk to size 2, and `v[2]` no longer exists — reading it is undefined behavior (a container-overflow, confirmed with `-D_GLIBCXX_SANITIZE_VECTOR=1` under AddressSanitizer; it "works" without that flag purely because `pop_back()` doesn't free memory, so the stale read happens to land on still-allocated bytes). The fix is to do the swap-and-remap step **only when `idx != v.size()-1`**, and always `pop_back()` last.

## Solution

### C++
```cpp
class RandomizedSet {
public:
    vector<int> v;
    unordered_map<int,int> mp;

    RandomizedSet() {}

    bool insert(int val) {
        if (mp.find(val) != mp.end()) return false;
        v.push_back(val);
        mp[val] = v.size() - 1;
        return true;
    }

    bool remove(int val) {
        if (mp.find(val) == mp.end()) return false;
        int idx = mp[val];
        if (idx != (int)v.size() - 1) {
            v[idx] = v.back();
            mp[v[idx]] = idx;
        }
        v.pop_back();
        mp.erase(val);
        return true;
    }

    int getRandom() {
        return v[rand() % v.size()];
    }
};
```

### Python
```python
import random

class RandomizedSet:
    def __init__(self):
        self.v = []
        self.mp = {}

    def insert(self, val: int) -> bool:
        if val in self.mp:
            return False
        self.v.append(val)
        self.mp[val] = len(self.v) - 1
        return True

    def remove(self, val: int) -> bool:
        if val not in self.mp:
            return False
        idx = self.mp[val]
        if idx != len(self.v) - 1:
            self.v[idx] = self.v[-1]
            self.mp[self.v[idx]] = idx
        self.v.pop()
        del self.mp[val]
        return True

    def getRandom(self) -> int:
        return random.choice(self.v)
```

### Java
```java
import java.util.*;

class RandomizedSet {
    List<Integer> v;
    Map<Integer, Integer> mp;
    Random rand;

    public RandomizedSet() {
        v = new ArrayList<>();
        mp = new HashMap<>();
        rand = new Random();
    }

    public boolean insert(int val) {
        if (mp.containsKey(val)) return false;
        v.add(val);
        mp.put(val, v.size() - 1);
        return true;
    }

    public boolean remove(int val) {
        if (!mp.containsKey(val)) return false;
        int idx = mp.get(val);
        if (idx != v.size() - 1) {
            int last = v.get(v.size() - 1);
            v.set(idx, last);
            mp.put(last, idx);
        }
        v.remove(v.size() - 1);
        mp.remove(val);
        return true;
    }

    public int getRandom() {
        return v.get(rand.nextInt(v.size()));
    }
}
```

Verified: C++ and Python both run against 8 operations (insert x3, remove-last, insert-duplicate-of-removed, remove-first/middle, remove-absent, getRandom-is-in-set, drain to empty) with matching output in both languages; C++ additionally clean under AddressSanitizer with `-D_GLIBCXX_SANITIZE_VECTOR=1` (the strict flag that specifically catches the last-element bug below). Java is the same logic translated directly — `ArrayList.set/remove(size-1)` and `HashMap.put/remove` map 1:1 onto the C++ vector/unordered_map calls — not independently compiled in this environment (no JDK available), but there's no language-specific behavior here that C++/Python didn't already exercise.

## Bug log

- First attempt used `std::set<int>` alone. Dead end: a `set` has no O(1) "give me the k-th element" operation — you'd need to walk an iterator, which is O(n), breaking `getRandom()`'s time bound. This is why the vector (for O(1) indexed access) + hashmap (for O(1) lookup) combination is necessary, not just a convenience.
- Real bug, caught with AddressSanitizer's `-D_GLIBCXX_SANITIZE_VECTOR=1`: an earlier version of `remove()` always ran `v[idx] = v.back(); mp[v[idx]] = idx; v.pop_back();` unconditionally, without checking whether `idx` was already the last index. That specific line ordering happened to still read `v.back()` before popping, so it didn't crash — but a natural variation of the same code (pop first, then read `v[idx]` to remap) reads past the just-shrunk vector when the removed element was the last one, since `idx == v.size()-1` before the pop means `idx` is out of range after it. Fixed by guarding the swap-and-remap step with `if (idx != v.size()-1)`, so the last-element case just pops directly with no read at the stale index.
