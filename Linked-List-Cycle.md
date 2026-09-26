# Linked List Cycle (LeetCode Easy)

## Problem

Given the head of a linked list, determine if the list has a cycle (some node's `next` eventually points back to a previously-visited node).

```
head = [3,2,0,-4], cycle back to index 1  -> true
head = [1,2], cycle back to index 0       -> true
head = [1]                                -> false
```

## How to explain it out loud

*"This is Floyd's cycle detection — two pointers moving through the list at different speeds. A slow pointer moves one step at a time, a fast pointer moves two steps at a time. If there's no cycle, the fast pointer just reaches the end first and I return false. If there is a cycle, the fast pointer eventually laps the slow pointer from behind — since it's gaining one step of distance on the slow pointer every iteration, and they're both stuck going around the same finite loop, they're guaranteed to land on the exact same node eventually. The moment they're equal, that's a cycle."*

## Approach — Floyd's cycle detection (slow/fast pointers)

Two pointers, both starting at `head`. Each iteration: advance `slow` by one node, `fast` by two nodes. Guard the loop with `fast != nullptr && fast->next != nullptr` — checking both is necessary since `fast` advances two steps at once and could otherwise dereference a null `next`. After advancing both, check `slow == fast`; if so, a cycle exists — return `true`. If the loop exits naturally (`fast` or `fast->next` hit `nullptr`), there's no cycle — return `false`.

Why this always terminates correctly: with no cycle, `fast` is strictly ahead of `slow` and reaches the end in bounded time. With a cycle, once both pointers are inside the loop, `fast` closes the gap on `slow` by exactly one node per iteration (it's going twice as fast around a fixed-size loop), so it cannot skip over `slow` — it's guaranteed to land exactly on `slow` within at most (cycle length) iterations.

Time: O(n) — in the worst case (cycle spanning the whole list), the pointers meet within O(n) steps, and with no cycle `fast` reaches the end in n/2 steps · Space: O(1) — no extra data structure, unlike a hash-set-of-visited-nodes approach which would also work but costs O(n) space.

## Solution

### C++
```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode* slow = head;
        ListNode* fast = head;

        while(fast!=NULL && fast->next!=NULL){
            slow = slow->next;
            fast = fast->next->next;
            if(slow==fast) return true;
        }
        return false;
    }
};
```

### Python
```python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = head
        fast = head
        while fast is not None and fast.next is not None:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                return True
        return False
```

### Java
```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return true;
        }
        return false;
    }
}
```

Verified against 9 cases: empty list, a single node with no cycle, a single node with a self-cycle, two nodes with no cycle, two nodes cycling back to the head, the classic 4-node LeetCode example (cycle to index 1), a 1000-node list with no cycle, a 1000-node list cycling back near the very end, and a 1000-node list forming one full loop back to the head — all correct, clean under AddressSanitizer (the only sanitizer flags were intentional, from deliberately not freeing the cyclic test lists in the harness — irrelevant to the algorithm itself, which never allocates). Python and Java are the same logic translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- Correct on the first attempt — no bugs found.
