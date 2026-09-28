# Binary Tree Right Side View (LeetCode Medium)

## Problem

Given the root of a binary tree, return the values visible when looking at the tree from the right side, ordered top to bottom (one value per level — whichever node is rightmost at that depth).

```
   1
  / \
 2   3
  \   \
   5   4

-> [1,3,4]
```

## How to explain it out loud

*"This is level-order BFS, and the right-side view is just 'the last node processed at each level' — since I always push left before right, popping in FIFO order means the last node popped in any level's batch is guaranteed to be that level's rightmost node, regardless of the tree's actual shape. So I do the standard size-snapshot BFS, and instead of collecting every value in the level, I just keep overwriting a `node` variable as I pop — whatever it holds when the inner loop finishes is this level's rightmost node, and that's what goes into the result."*

## Approach — BFS, keep the last node popped per level

Standard level-order BFS with a queue, seeded with `root`. Each outer iteration: snapshot `size = q.size()` before draining anything (this fixes exactly how many nodes belong to the current level, same reasoning as level-order traversal). Loop `size` times: pop a node into a local variable `node` (overwriting it each time), push its non-null children (left before right, so children stay properly ordered left-to-right in the queue for the next level). After the inner loop, `node` holds whichever node was popped *last* — since nodes are popped in left-to-right order within a level, that's guaranteed to be the rightmost node at this depth. Push its value into the result.

Why this correctly handles trees where the right side is "shorter" than the left (e.g. a right subtree ending early while the left keeps going): the algorithm never assumes the rightmost *child pointer* path is what's visible — it works purely from "which node was rightmost *at this actual level*," which is exactly the correct definition of the right-side view even when the tree is lopsided.

Time: O(n) — every node pushed and popped once · Space: O(w) for the queue (widest level), plus O(h) for the output where `h` is the tree height.

## Solution

### C++
```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> result;
        if(!root) return result;

        queue<TreeNode*> q;
        q.push(root);
        
        while(!q.empty()){
            TreeNode* node = q.front();
            int size = q.size();
            for(int i = 0; i<size; i++){
                node = q.front();
                q.pop();
                if(node->left) q.push(node->left);
                if(node->right) q.push(node->right);
            }
            result.push_back(node->val);
        }
        return result;
    }
};
```

### Python
```python
from collections import deque

class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> list[int]:
        result = []
        if not root:
            return result

        q = deque([root])
        while q:
            size = len(q)
            node = None
            for _ in range(size):
                node = q.popleft()
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            result.append(node.val)

        return result
```

### Java
```java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);

        while (!q.isEmpty()) {
            int size = q.size();
            TreeNode node = null;
            for (int i = 0; i < size; i++) {
                node = q.poll();
                if (node.left != null) q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }
            result.add(node.val);
        }
        return result;
    }
}
```

Verified against 6 hand-picked cases: the classic LeetCode example, a right-skewed 2-node tree, an empty tree, a single node, a complete 7-node tree, and the tricky case where a left subtree extends one level deeper than the right side (confirming the view correctly reports the deepest-left node at that extra level, not a stale rightmost value) — all correct, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same "keep the last node popped per level" BFS translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- Correct on the first attempt — no bugs found.
