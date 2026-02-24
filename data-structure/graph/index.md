# Graph

### Why Graphs Are Transformational

- Arrays → Linear structure
- Trees → Hierarchical structure
- Graphs → Arbitrary structure

Real systems are graphs:
- Microservices dependencies
- Package dependency trees (actually DAGs)
- Social networks
- Routing systems
- State machines
- Workflows
- Compiler dependency resolution
- Kubernetes pod communication
- Railway network (😉 your RailPulse future)

## What is graph?

> A collection of entities (nodes) and relationships (edges).

### Types of graph

1️⃣ Directed vs Undirected
- Undirected → friendship
- Directed → follower

2️⃣ Weighted vs Unweighted
- Unweighted → each edge cost = 1
- Weighted → edges have costs

3️⃣ Cyclic vs Acyclic
- Cycles exist?
-No cycles? → DAG

### How graphs are stored in C++?

Two representations -

#### Adjacency List

```
int n;
vector<vector<int>> adj(n);
```

Memory: O(V + E)
Best for most real problems.

#### Adjacency Matrix

```
vector<vector<int>> matrix(n, vector<int>(n));
```

Memory: O(n²)
Used rarely unless dense graph.

### The Two Core Graph Traversals

Everything reduces to:
- BFS
- DFS

But now:
- Multiple components
- Cycles
- No guaranteed root

Which means:

> We must track visited nodes.

## Problems

### First

> Given n nodes and edges, count number of connected components.

```
0 - 1    2 - 3
    |
    4
```
Components = 2

#### Intuition (don’t code yet)

Algorithm:
- Maintain visited[]
- Loop through all nodes:
  - If not visited:
    - Run DFS/BFS
    - Increment component count

This teaches:
- Graphs may not be fully connected
- We need outer loop + traversal

#### Mental Model

In trees:
- Start at root

In graphs:
- There may be multiple “roots”

That’s structural thinking shift.

### Second

> Suppose: Graph has 1 million nodes, 10 million edges. Which traversal is more memory-heavy: BFS/DFS And what determines the answer? Think in terms of: Graph shape, Not just algorithm label

**Case 1** : Start Graph

```
      2
      |
3 — 1 — 4 — 5 — ...
      |
      6
```

**DFS**:
- Visit 1
- Visit 2 (return)
- Visit 3 (return)
- Visit 4 (return)
- ...
- Depth never exceeds 2

Memory ≈ O(2)

**BFS**:
- Visit 1
- Push 1,000,000 neighbors into queue

Queue size ≈ 1,000,000

Memory ≈ O(n)

👉 In this case, BFS is more memory-heavy.

**Case 2**: Long Chain

```
1 — 2 — 3 — 4 — ... — 1,000,000
```

DFS:
- Depth = 1,000,000
- Stack overflow risk

BFS:
- Queue size never > 2

👉 Here, DFS is more memory-heavy.

🔥 So What Actually Determines Memory?
- Not density.
- Not number of edges.

The real factor is:

> The graph’s maximum breadth vs maximum depth.

Exactly like trees.

| Traversal | Memory depends on               |
| --------- | ------------------------------- |
| DFS       | Longest path depth              |
| BFS       | Maximum frontier size (breadth) |


> It depends on graph structure. 
> 
> Deep graph → DFS heavy
> 
> Wide graph → BFS heavy

### Third

> Suppose you have:
> 
> A complete graph with 1 million nodes
(every node connected to every other node)
>
> Which traversal explodes in memory first?
>
> And why?

📌 Complete Graph (1 million nodes)

Every node connected to every other node.

Edges ≈ 10¹² scale conceptually (very dense).

🔵 BFS Behavior

Start from node 0.
- Mark node 0 visited.
- Push all its neighbors into queue.

Neighbors = 999,999 nodes.

Queue size ≈ 999,999 immediately.

That’s O(n).

After processing them, queue shrinks —
but the peak memory already happened.

So yes:

> BFS frontier explodes immediately.

🟢 DFS Behavior

Start at node 0.

DFS picks one neighbor and goes deep.

Because every node connects to every other node:
- From node 0 → node 1
- From node 1 → node 2
- From node 2 → node 3

…

We’ll keep going deeper until all nodes are visited.

So recursion depth can reach ≈ 1,000,000.

That’s also O(n).

#### Conclusion

In a complete graph,

| Traversal | Peak Memory |
| --------- | ----------- |
| BFS       | O(n)        |
| DFS       | O(n)        |

Both can explode.

The difference is how they explode:
- BFS: large frontier immediately
- DFS: deep call stack gradually

Graph density does not automatically mean DFS is safe.

The deciding factors are:
- Maximum path depth
- Maximum frontier width

In a complete graph:
- Depth ≈ n
- Breadth ≈ n

Both bad.

Memory cost depends on:

> max(depth, frontier size)

Whichever is larger dominates.

### Fourth

> Suppose the graph is:
>
> A balanced binary tree structure (as a graph)
>
> 1 million nodes
>
> Which traversal has larger memory footprint?


For 1 million nodes:
- Height ≈ log₂(1e6) ≈ 20
- Bottom level contains ~500,000 nodes

BFS processes level by level.

At the last level:
- ~500,000 nodes are in the queue at once.

So memory ≈ O(n)

DFS goes down one branch:
- Depth ≈ 20
- Call stack size ≈ 20

Memory ≈ O(log n)

#### Conclusion

So, for balanced graph -

| Traversal | Memory   |
| --------- | -------- |
| BFS       | O(n)     |
| DFS       | O(log n) |

DFS is dramatically more memory efficient here.

Traversal memory depends on which dimension dominates:
- Balanced tree → breadth dominates → BFS heavy
- Skewed tree → depth dominates → DFS heavy
- Complete graph → both dominate → both heavy
- Star graph → breadth dominates → BFS heavy

We’re no longer choosing traversal by name.

We’re choosing by shape analysis.


### Fifth

> In real-world systems:
>
> Social networks → extremely wide graphs
>
> Dependency chains → often deep graphs
>
> File systems → mixed
>
> If you were writing production code for unknown graph shapes,
which traversal would you prefer by default:
>
> - Recursive DFS
> - Iterative DFS
> - BFS
>
> And why?

1️⃣ Recursive DFS
- Memory = O(depth)
- Risk = stack overflow
- Depth could be N in worst case
- System stack is limited

So yes — risky for unknown graph shape.

2️⃣ Iterative DFS (explicit stack)
- Memory = O(depth)
- Same depth behavior as recursive
- But stack lives on heap → no stack overflow
- You control memory

Important:
> Iterative DFS does NOT reduce memory.
> 
> It just moves memory from system stack to heap.

If graph is a long chain:
- Depth = 1,000,000
- Stack size = 1,000,000 entries

Still O(N).

3️⃣ BFS
- Memory = O(frontier size)
- Worst case = O(N)

#### Conclusion
🔥 So What’s The Real Safe Default?

For unknown graph shape:

❌ Recursive DFS → unsafe (stack overflow risk)

❌ BFS → can explode in wide graphs

✅ Iterative DFS → safest

| Traversal     | Worst Case Memory             |
| ------------- | ----------------------------- |
| Recursive DFS | O(N) (stack, unsafe)          |
| Iterative DFS | O(N) (heap, safe)             |
| BFS           | O(N) (heap, depends on width) |
