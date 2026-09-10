# Add Two Numbers (LeetCode Medium)

## Problem

Two non-empty linked lists represent two non-negative integers, with digits stored in **reverse order** (the 1's digit is the head). Add the two numbers and return the sum as a linked list, in the same reverse-digit format.

```
l1 = [2,4,3]   (represents 342)
l2 = [5,6,4]   (represents 465)
-> [7,0,8]     (342 + 465 = 807)
```

## How to explain it out loud

*"Digits are stored least-significant-first, which maps directly onto how addition works by hand — process one digit position at a time, front to back, carrying into the next position when a column sums past 9. Walk both lists together: at each step add the two current digits (treating a list that's run out as contributing 0) plus whatever carry came from the previous step, the new digit is that sum mod 10, and the new carry is that sum divided by 10. Keep going as long as either list still has nodes, or there's still a carry to place — after both lists are fully consumed, if there's a leftover carry, it needs one more node of its own, since it represents a new highest digit."*

## Approach

Handle the first digit pair directly, then loop over the rest. Keep a pointer to the very first node created (`ans`, the value you'll return) separate from a second pointer (`head`) that walks forward as new nodes get attached — this is essential, since you need to both build the list forward and still return its true starting point afterward.

At each step past the first pair: take the current digit from `l1` if it still has nodes (otherwise treat it as `0`), same for `l2`, add both plus the running `carry`. New digit = that sum `% 10`; new carry = that sum `/ 10`. Advance whichever of `l1`/`l2` still has nodes. Continue while *either* list still has remaining nodes.

After the loop, if `carry` is still nonzero, it represents one more digit that hasn't been placed anywhere yet — attach one final node holding it.

Time: O(max(len(l1), len(l2))) · Space: O(max(len(l1), len(l2))) for the output list

## Solution

### C++
```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        if (l1 == NULL) return l2;
        if (l2 == NULL) return l1;

        ListNode* ans = new ListNode((l1->val + l2->val) % 10);
        ListNode* head = ans;
        int carry = (l1->val + l2->val) / 10;
        l1 = l1->next;
        l2 = l2->next;

        while (l1 != NULL || l2 != NULL) {
            ListNode* temp = new ListNode();
            temp->val = (carry + (l1 ? l1->val : 0) + (l2 ? l2->val : 0)) % 10;
            carry = (carry + (l1 ? l1->val : 0) + (l2 ? l2->val : 0)) / 10;
            if (l1) l1 = l1->next;
            if (l2) l2 = l2->next;
            head->next = temp;
            head = head->next;
        }

        if (carry) head->next = new ListNode(carry);
        return ans;
    }
};
```

### Python
```python
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        if l1 is None:
            return l2
        if l2 is None:
            return l1

        ans = ListNode((l1.val + l2.val) % 10)
        head = ans
        carry = (l1.val + l2.val) // 10
        l1 = l1.next
        l2 = l2.next

        while l1 is not None or l2 is not None:
            v1 = l1.val if l1 else 0
            v2 = l2.val if l2 else 0
            temp = ListNode((carry + v1 + v2) % 10)
            carry = (carry + v1 + v2) // 10
            if l1: l1 = l1.next
            if l2: l2 = l2.next
            head.next = temp
            head = head.next

        if carry:
            head.next = ListNode(carry)
        return ans
```

### Java
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        ListNode ans = new ListNode((l1.val + l2.val) % 10);
        ListNode head = ans;
        int carry = (l1.val + l2.val) / 10;
        l1 = l1.next;
        l2 = l2.next;

        while (l1 != null || l2 != null) {
            int v1 = (l1 != null) ? l1.val : 0;
            int v2 = (l2 != null) ? l2.val : 0;
            ListNode temp = new ListNode((carry + v1 + v2) % 10);
            carry = (carry + v1 + v2) / 10;
            if (l1 != null) l1 = l1.next;
            if (l2 != null) l2 = l2.next;
            head.next = temp;
            head = head.next;
        }

        if (carry != 0) head.next = new ListNode(carry);
        return ans;
    }
}
```

Verified against 6 hand-picked cases (the classic example, a single-digit carry overflow, a longer-list carry propagating through multiple positions, matching single-digit lists with no carry, a carry cascading through three consecutive 9s, and mismatched-length lists with no carry) plus a 1000-trial randomized stress test against an independent big-number-addition reference (interprets both lists as reversed-digit integers, adds them directly, converts back to a digit list) — 0 mismatches. C++ clean under AddressSanitizer (leak detection excluded — the disposable test harness and the intentionally-returned linked list nodes aren't freed, unrelated to the solution's correctness). Python matches on all 6 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- First attempt had two stacked syntax typos that prevented compilation: `ListNode* ans - new ListNode();` (a stray `-` where `=` was intended), and `carry = (...)\10;` in three places (a backslash `\` where integer-division `/` was intended).
- After fixing those typos, a deeper structural bug remained: no new node was ever created for any digit beyond a dummy placeholder (`ans->next` was never assigned), so `ans = ans->next` immediately advanced to a null pointer, and the function also returned `ans` itself after it had been walked forward through the whole list — meaning even a correctly-built list would have returned a pointer to its *last* node rather than its first.
- A revised attempt fixed node creation (allocating a `temp` node per digit) but kept the same "return the walking pointer" bug, confirmed concretely: `[2,4,3]+[5,6,4]` returned `[8]` instead of `[7,0,8]`, and separately, `[5]+[5]` returned `[0]` instead of `[0,1]` because a leftover final carry was never converted into its own node.
- Final version fixes both: keeps `ans` fixed at the true first node (returned at the end) while a separate `head` pointer walks forward and attaches new nodes, and adds an explicit `if (carry) head->next = new ListNode(carry);` check after the main loop to place any final overflow digit. Verified against 1000 random trials with 0 mismatches.
