# Binary Tree Level Order Traversal (LeetCode Medium)

## Problem

Given the root of a binary tree, return the values grouped level by level, top to bottom, left to right within each level.

```
      3
     / \
    9  20
       / \
     15   7

-> [[3],[9,20],[15,7]]
```

## How to explain it out loud

*"This is BFS with a queue, but the key detail is knowing exactly where one level ends and the next begins. Right before I start draining the current level, I snapshot the queue's size into a local variable — that's exactly how many nodes belong to this level, since every node currently sitting in the queue was pushed by the previous level's processing, before any of this level's own children get pushed. I loop exactly that many times, popping a node, recording its value, and pushing its children — and because I looped against the *snapshot*, not the queue's live, constantly-changing size, the children I just pushed don't get swept into this same level by mistake."*

## Approach — BFS, level-size snapshot

Standard BFS with a queue, seeded with `root`. Each outer-loop iteration processes exactly one level: capture `size = q.size()` into a local variable *before* touching the queue this round. Run the inner loop exactly `size` times — not `q.size()` re-checked live — popping a node, recording its value into `level`, and pushing its non-null children (these get appended after everything already in the queue, so they're correctly deferred to the *next* level, not this one). After the inner loop, push `level` into the result.

Time: O(n) — every node pushed and popped once · Space: O(w) for the queue (widest level, worst case O(n)), plus O(n) for the output.

## Solution

### C++
```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;
        if(!root) return result;

        queue<TreeNode*> q;
        q.push(root);
        while(!q.empty()){
            vector<int> level;
            int size = q.size();
            for(int i=0; i<size; i++){
                TreeNode* node = q.front();
                q.pop();
                level.push_back(node->val);
                if(node->left) q.push(node->left);
                if(node->right) q.push(node->right);
            }
            result.push_back(level);
        }
        return result;
    }
};
```

### Python
```python
from collections import deque

class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> list[list[int]]:
        result = []
        if not root:
            return result

        q = deque([root])
        while q:
            level = []
            size = len(q)
            for _ in range(size):
                node = q.popleft()
                level.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            result.append(level)

        return result
```

### Java
```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);

        while (!q.isEmpty()) {
            int size = q.size();
            List<Integer> level = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = q.poll();
                level.add(node.val);
                if (node.left != null) q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }
            result.add(level);
        }
        return result;
    }
}
```

Verified against 4 hand-picked cases (the classic LeetCode example, a single node, an empty tree, and a complete 7-node tree spanning 3 full levels) — all correct, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same level-size-snapshot BFS translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- First attempt used `for(int i=0; i<q.size(); i++)` — checking `q.size()` fresh on every iteration instead of snapshotting it once before the loop. Confirmed as a real bug by direct execution: since children get pushed into the same queue *inside* that loop, `q.size()` grows mid-loop, so the loop doesn't stop after processing exactly one level's worth of nodes — it keeps going and pulls in nodes that belong to the *next* level too. On the classic example `[3,9,20,null,null,15,7]`, this produced `[[3,9],[20,15],[7]]` instead of the correct `[[3],[9,20],[15,7]]` — levels 1 and 2 got incorrectly merged, splitting what should've been three balanced levels into a differently-shaped (but same total node count) grouping. Fixed by capturing `int size = q.size()` once, right before the inner loop, and looping against that fixed snapshot instead of the live, constantly-changing queue size — the same pattern already used correctly in Average of Levels in Binary Tree. Re-verified afterward against all 4 test cases.
