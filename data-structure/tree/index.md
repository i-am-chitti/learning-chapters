# Tree

Arrays taught sequence.
Trees teach structure.

#### What a tree really represents

A tree models:
- File systems
- Org hierarchies
- JSON objects
- DOM
- Dependency graphs
- Routing trees
- Decision trees

### Tree Traversal

#### DFS — Depth First Search

> Go deep before wide.

Pattern:
- Use recursion OR stack
- Process node
- Go left
- Go right

We explore one branch fully.

#### BFS - Breadth First Search

> Explore level by level.

Pattern:
- Use queue
- Process level
- Add children
- Continue

We explore layer by layer.

#### Visual

```
        1
       / \
      2   3
     / \
    4   5
```

DFS order (preorder):

1 → 2 → 4 → 5 → 3

BFS order:

1 → 2 → 3 → 4 → 5

### When to use DFS vs BFS

| Situation                       | Use |
| ------------------------------- | --- |
| Need shortest path (unweighted) | BFS |
| Need to explore all paths       | DFS |
| Need level info                 | BFS |
| Need subtree properties         | DFS |
| Backtracking                    | DFS |


### Code

#### Tree Structure
```
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;

    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

#### DFS
```
void dfs(TreeNode* root) {
    if (!root) return;

    cout << root->val << " ";
    dfs(root->left);
    dfs(root->right);
}
```

#### BFS
```
void bfs(TreeNode* root) {
    if (!root) return;

    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        TreeNode* node = q.front();
        q.pop();

        cout << node->val << " ";

        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```

### Maximum Depth of Tree

> If we want to compute the maximum depth of a binary tree,
would we prefer DFS or BFS?
And why?

DFS maps very cleanly to:

“Depth = 1 + max(left_depth, right_depth)”

BFS is capable too. Two ways:

1. Process level by level and increment depth after each level.
2. Store (node, depth) in the queue.

**Difference**

The deeper difference is not capability — it’s natural fit.

DFS aligns with the recursive definition:

```
depth(node) = 1 + max(depth(left), depth(right))
```

So DFS feels:
- Direct
- Mathematical
- Elegant

**BFS aligns with:**

> “Count how many levels exist.”

That’s also clean:
- Each layer processed → depth++
- When queue empties → depth is known

Both works in O(n).
The real question is:
> Which traversal mirrors the definition of the problem?

For maximum depth:
- The recursive definition mirrors DFS naturally.

So DFS is conceptually cleaner.

### Height and depth

> If the tree is extremely skewed (like a linked list of 100,000 nodes),
which approach is safer in C++ — DFS or BFS?
And why?

BFS is safer.

In a skewed tree:
- At any time, queue holds at most:
    - current node
    - possibly one child

So queue size ≈ 1

Memory: O(1) (effectively)

**DFS issue**

Recursive DFS does:
- Each recursive call pushed to call stack
- Depth = 100000
- Stack frames = 100000

This will likely cause: Stack Overflow

> Recursive DFS is unsafe for very deep trees.

#### Important Nuance

DFS itself is not unsafe.

**Recursive DFS is unsafe.**

We can write DFS iteratively using an explicit stack:

```
int maxDepth(TreeNode* root) {
    if (!root) return 0;

    stack<pair<TreeNode*, int>> st;
    st.push({root, 1});
    int maxDepth = 0;

    while (!st.empty()) {
        auto [node, depth] = st.top();
        st.pop();

        maxDepth = max(maxDepth, depth);

        if (node->left) st.push({node->left, depth + 1});
        if (node->right) st.push({node->right, depth + 1});
    }

    return maxDepth;
}
```

Now:
- Memory is also O(height)
- But controlled explicitly
- No risk of stack overflow

> Implicit stack vs explicit memory control

Recursive DFS:
- Uses system stack
- Limited
- Risky for deep inputs

Iterative DFS / BFS:
- Uses heap memory
- Safer for large structures

#### Rule

| Structure     | Memory Usage      |
| ------------- | ----------------- |
| Recursive DFS | O(height) (stack) |
| Iterative DFS | O(height) (heap)  |
| BFS           | O(width)          |


So:
- Skewed tree → BFS safer
- Wide tree (like complete binary tree) → DFS safer

Because in a wide tree:
- BFS queue can grow to O(n/2)

#### Question

> Consider a complete binary tree with 1 million nodes. Which traversal is more memory-heavy: DFS/BFS. And why?

DFS memory = height ≈ logN

BFS memory = width ≈ 2^d

| Traversal | Memory usage |
| --------- | ------------ |
| DFS       | O(height)    |
| BFS       | O(width)     |


For a complete tree:
- Height ≈ log N
- Width ≈ N / 2

```
log N  <<  N
```

DFS is dramatically more memory efficient here.


The correct way to think is not:

> "Which algorithm is better?"

But:

> Which dimension of the tree explodes? Height or width?

Because traversal memory expands along one dimension.


Tree Memory Intuition Rule
- Tall & skinny tree → DFS dangerous (stack overflow risk)
- Wide & shallow tree → BFS dangerous (queue explosion risk)

We must analyze shape first, then choose traversal.

> Traversal cost depends more on tree shape than algorithm name.


#### Question

> Suppose: We must compute minimum depth of a binary tree. The tree is extremely wide but shallow. The first leaf is very close to root. Which traversal is computationally smarter?

BFS is smarter because we can early terminate when we hit the first leaf at the closest level.

Why DFS is less efficient here?

It might:
- Go very deep on one side
- Completely miss a shallow leaf on the other side

Unless we carefully prune, DFS:
- Might explore unnecessary deep paths
- Does more work before discovering the minimum

> BFS guarantees the first leaf encountered is the minimum depth leaf.

#### Insight

| Goal                         | Best traversal |
| ---------------------------- | -------------- |
| Shortest path                | BFS            |
| Longest / aggregate property | DFS            |
| Explore all paths            | DFS            |
| Level-based processing       | BFS            |


#### Master approach
If the problem contains the word:
- “Shortest”
- "Minimum steps”
- “Closest”
- “Nearest”

Our brain should immediately whisper:
> BFS.

If it contains:
- “All paths”
- “Maximum”
- “Subtree”
- “Backtracking”

Our brain should whisper:
> DFS.

## Tree <-> Graph

> Suppose you have an unweighted graph (not a tree). You want the shortest path from node A to node B. Would our answer change? And why?

To find the shortest path, we must explore all paths. Is this really necessary? OR Is there a traversal order that guarantees
the first time you reach B is already optimal?


In a tree, we said:
- BFS explores level by level
- First leaf found = minimum depth

Now imagine a graph like this:

```
A -- C -- D -- B
 \ 
  E -- B
```

There are multiple paths from A to B.

Instead of thinking “paths”, think:

> How many edges away is each node from A?

**What Does DFS Actually Guarantee?**

DFS explores like this:
```
A → C → D → B
```

Suppose that path length = 3

But what if there was:
```
A → E → B
```

Length = 2

If DFS chooses C first, what happens?

Will it discover the length-2 path before or after finishing the deep one?

Think about order.

**Clue**

In an unweighted graph:
- All edges have equal cost (1).
- So shortest path = minimum number of edges.

Now ask ourselves:

Which traversal explores nodes in increasing number of edges from the source?

> In an unweighted graph, BFS guarantees shortest path because BFS expands strictly in order of edge count.

DFS has:
- No distance ordering
- No layered guarantee

So it must explore all possible paths to be certain.