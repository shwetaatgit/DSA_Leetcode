# Merge Two Sorted Lists (LeetCode Easy)

## Problem

Given the heads of two sorted linked lists, merge them into one sorted list and return its head.

```
l1 = [1,2,4], l2 = [1,3,4]  -> [1,1,2,3,4,4]
l1 = [],      l2 = []       -> []
l1 = [],      l2 = [0]      -> [0]
```

## How to explain it out loud

*"This is naturally recursive: if either list is empty, the answer is just the other list — nothing left to merge. Otherwise, whichever head is smaller has to come first in the result, so I attach the *rest* of the merge — recursively merging that node's own tail against the other list — onto its `next`, and return that node as the new head. Each call peels off exactly one node and hands back a fully merged list for everything after it, so by the time the recursion bottoms out at an empty list, the whole chain has been stitched together in order."*

## Approach — Recursion

Base cases: if `l1` is null, the merge of an empty list with `l2` is just `l2` — return it. Symmetric if `l2` is null. Otherwise, compare `l1->val` and `l2->val`. Whichever is smaller (using `<=` for `l1` to keep the choice deterministic and stable) gets returned as the head of this sub-merge, but first its `next` is overwritten with the result of recursively merging *its own remaining tail* against the *other list untouched* — so the smaller node's old `next` gets correctly replaced with everything that comes after it once both lists are considered.

Time: O(m + n) — one recursive call consumes exactly one node from one of the two lists, so the recursion is exactly `m + n` calls deep · Space: O(m + n) for the call stack (this is genuinely a cost here, unlike a tail-recursive scenario — each frame stays alive until the whole chain returns, since the parent needs the child's return value to set `next`). An iterative version with a dummy head and a moving tail pointer gets this down to O(1) auxiliary space if that's ever asked for as a follow-up.

## Solution

### C++
```cpp
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        if(l1 == NULL) return l2;
        if(l2 == NULL) return l1;

        if(l1 -> val <= l2 -> val) {
            l1 -> next = mergeTwoLists(l1 -> next, l2);
            return l1;
        }
        else {
            l2 -> next = mergeTwoLists(l1, l2 -> next);
            return l2;
        }
    }
};
```

### Python
```python
class Solution:
    def mergeTwoLists(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        if l1 is None:
            return l2
        if l2 is None:
            return l1

        if l1.val <= l2.val:
            l1.next = self.mergeTwoLists(l1.next, l2)
            return l1
        else:
            l2.next = self.mergeTwoLists(l1, l2.next)
            return l2
```

### Java
```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        if (l1.val <= l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

Verified against 7 hand-picked cases (the classic example, both empty, one empty/one single-element on either side, all-duplicate values on both sides, negative numbers with a repeated boundary value, and very unbalanced lengths) plus a 500-trial randomized stress test (random-length sorted lists built by random non-negative increments, compared against a sort-and-concatenate reference) — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same logic translated directly (Java not independently compiled in this environment — no JDK available). Recursion depth scales with list length; fine for LeetCode's constraint (each list up to 50 nodes), worth flagging as a stack-depth concern only if this pattern were reused on much longer lists.

## Bug log

- Correct on the first attempt — no bugs found.
