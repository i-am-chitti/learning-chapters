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

> Suppose we have:
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
> If we were writing production code for unknown graph shapes,
which traversal would we prefer by default:
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
- We control memory

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

### Cycles in undirected graph

> Suppose we are solving:
>
> Detect if an undirected graph contains a cycle.
>
> Would we prefer:
>
> - BFS
> - DFS
>
> And why?

Given this adjacency list - 

```
1 -> 2 
2 -> 1,3,4 
3 -> 2,4 
4 -> 2,3,5 
5 -> 4
```
Here a cycle exists => 2 → 3 → 4 → 2

Confusion we'll exist:
> In DFS, how do I differentiate between a normal visited edge and a real cycle?

When we move from 3 -> 4, which node did we come from? => 2

2 is our parent.

Now:

> If during DFS, we see a visited node that is NOT our parent,
what does that imply?
>
> That implies that we detected a cycle.
>
> When exploring from node `u` to neighbor `v`:
>
> If `v` is visited AND `v` is not parent of `u`,
what structural situation must exist?

that is a closed loop. There must be a cycle existing between `u` and `v`.

> In an undirected graph, 
> If during DFS,
we encounter a visited node that is not our parent,
>
> then:
>
> There already exists a path from that node to we through DFS tree,
and the new edge closes the loop.


During DFS: For each neighbor v of u:
- If v is not visited → recurse.
- If v is visited AND v != parent → cycle detected.

### Cycles in directed graph

> In a directed graph, the rule is different.
>
> Why?

In a directed graph, DFS with single parent can't work because traversing path matters here.

We must track the current path from source node to target node and only if this path leads to already visited node, then, it's a cycle.
By visited, I meant presence of node in current path or in recursion stack.

> DFS + recursion stack state is needed to detect cycles in a directed graph.

🔥 What “recursion stack” really means

When we run DFS:
- we go from A → B → C → D
- These nodes are currently in the recursion stack.
- This represents an active path from root to current node.

So the recursion stack is literally:

> The current path from start node to current node.

🚫 Why “visited” alone fails

If a node is:
- Visited
- Fully processed
- Not in recursion stack

Then there is no active path from it to us.

So an edge to it does NOT form a cycle.

Because there is no way to return back.

#### Rule

During DFS:

For neighbor v of u:
- If v not visited → recurse
- If v is in recursion stack → cycle
- If v visited but not in stack → ignore

**🧠 Let’s Cement This With a Mental Model**

Think of DFS stack as:

> A path we are currently walking.

If we see an edge that connects back to the path we’re walking on,
we just formed a loop.

If we see an edge to somewhere we already left and finished,
that path is closed — it cannot form a loop.

#### Question

> In directed graph cycle detection,
> 
> why can’t we just use BFS instead of DFS?

Because:
- DFS naturally maintains the current path (recursion stack).
- BFS naturally maintains a frontier (levels), not a path.


**🧠 Why DFS Works Naturally for Directed Cycle Detection**

In directed graphs, a cycle means:

> There exists a path from a node back to one of its ancestors in the current path.

DFS gives us:
- A single active path from root → current node
- That path is explicitly represented by recursion stack

So when we encounter a node already in the recursion stack:
- we found an edge pointing backward along the current path
- That closes a directed loop

DFS is path-oriented.

**🧠 Why BFS Is Not Natural Here**

BFS explores level-wise.

It does not maintain:

> “The exact path from source to current node.”

Instead, it maintains:

> “All nodes at distance d.”

There is no natural notion of “ancestor in current path.”

So detecting a back-edge using BFS is not straightforward.

We would have to artificially maintain path history per node — which defeats BFS simplicity.

#### Conclusion

- If the problem is about reachability layers or minimum steps → BFS

- If the problem is about path properties or structural loops → DFS

#### Alternative algorithm - Kahn's Algorithm

> We detect cycle in directed graph using DFS + recursion stack.
>
> But there exists another algorithm for detecting cycle in directed graph that does NOT use DFS recursion stack.
>
> It is based on:
>
>Removing nodes with indegree 0 repeatedly.
>
> Can we guess what type of graph that algorithm applies to?

Indegree - Number of edges pointing into a node.

Outdegree - Number of edges going out of a node.


**Indegree = 0 means :**
- No one points to it.
- It has no prerequisites.
- Nothing must come before it.

> If nodes remain after repeatedly removing indegree-0 nodes, they form a cycle. If there exists a cycle, indegree can never be 0.


> Every node in the cycle has at least one incoming edge from another node in the same cycle.

**🔥 Structural Upgrade**

We now have two independent ways to detect directed cycles:

1️⃣ DFS + recursion stack

→ Detects back edge (path-based logic)

2️⃣ Indegree removal (Kahn’s Algorithm)

→ Detects dependency deadlock (dependency-based logic)


| Method           | Perspective           |
| ---------------- | --------------------- |
| DFS              | Path existence        |
| Indegree removal | Dependency resolution |

##### Question

> Suppose we run the indegree-removal algorithm and:
>
> we successfully remove all nodes.
>
> What does that guarantee about the graph?

If all nodes can be removed using indegree-0 elimination, the graph doesn't have any cycle. The graph is a **DAG (Directed Acyclic Graph)**.

> So graph is either single component or multiple split components.

This is incorrect.

Component structure is unrelated here.

A graph can:
- Be a single connected component and still be a DAG.
- Be multiple disconnected components and still be a DAG.
- Be single component and cyclic.
- Be multiple components and cyclic.

Connectivity and acyclicity are independent properties.

#### Remember

Undirected cycle detection

→ DFS + parent check

Directed cycle detection

→ DFS + recursion stack
OR
→ Indegree elimination (Kahn’s algorithm)

### Kahn's Algorithm

Kahn’s algorithm is not just cycle detection.

It is actually:

> A topological sort.

Meaning:

If the graph is a DAG,
we can produce an ordering such that:

For every edge:

```
u → v
```

u appears before v.

That’s dependency resolution.

This shows up in:
- Build systems
- Package managers
- Task scheduling
- CI pipelines
- Course prerequisite problems
- Microservice startup order

#### Real world

> Suppose we have: 5 tasks
>
> Edges represent prerequisites
>
> If topological sort is possible:
>
> What does that guarantee about task scheduling?

It means tasks form directed acyclic graph - don't have any cycle. There is ordering in task. Task u must be completed before task v.

#### 🔥 What Topological Sort Really Guarantees

If topological ordering exists:

**1️⃣ No Circular Dependencies**

We cannot have:

```
A depends on B
B depends on C
C depends on A
```

Because then no task can start.

That’s a deadlock.

So DAG ⇒ no dependency deadlock.

**2️⃣ There Exists At Least One Valid Execution Order**

Not necessarily unique.

Example:

```
A → C
B → C
```

Possible valid orders:
- A, B, C
- B, A, C

So topological order is about valid scheduling, not unique ordering.

**3️⃣ Tasks Can Be Parallelized**

If two nodes both have indegree 0 at the same time:

They can run in parallel.

🧠 Structural Interpretation

Graph view:
- Nodes = tasks
- Directed edge u → v = u must finish before v

Topological sort view:
- Find an order respecting dependencies.

Cycle view:
- Circular dependency = impossible schedule.

**Topological sort is:**

> Dependency resolution.

This shows up everywhere in engineering:
- Docker build layers
- Terraform dependency graph
- Kubernetes init order
- Makefile build graph
- CI job pipelines
- Package installation order

#### Topological Sorting

> Suppose:
>
> We are given n tasks and prerequisite pairs.
>
> If multiple tasks have indegree 0 at the same time,
>
> What does that tell us about:
>
> The uniqueness of topological ordering?
>
> The structure of the graph?


If multiple tasks have indegree 0, they can run in parallel. There will be multiple topological orderings possible.

🔥 When Is Topological Order Unique?

Here is the key structural fact:

> A DAG has a unique topological ordering
iff at every step there is exactly one node with indegree 0.

That’s the entire rule.

**🧠 Intuition: Why Multiple Indegree-0 Nodes Break Uniqueness**

Suppose at some step:

```
Available nodes: A, B
```

Both have indegree 0.

You can pick:

- A first, then B
or
- B first, then A

Both produce valid schedules.

That immediately gives at least 2 valid topological orders.

So:

> If at any step you have a choice, ordering is not unique.

**🧠 When Is It Unique?**

Imagine this graph:

```
1 → 2 → 3 → 4
```

At every step:
- Only one node has indegree 0.

No choice.
No branching.

Therefore:
Topological order must be:

```
1, 2, 3, 4
```

Only one possible.

**🔐 Structural Interpretation**

Uniqueness happens when:

> The DAG forms a single forced chain of dependencies.

In graph theory terms:

The DAG behaves like a linear ordering.

**🔥 Real-World Interpretation**

If our task graph has:

- Multiple independent tasks at the same time → flexible schedule.
- Exactly one executable task at each moment → rigid dependency chain.

Rigid chain ⇒ unique ordering.

Flexible graph ⇒ multiple valid orderings.

#### How to deduce topological ordering given a graph?

Let’s outline the algorithm clearly (Kahn’s algorithm intuition form):

1. Compute indegree of all nodes.
2. Put all nodes with indegree 0 into a queue.
3. While queue not empty:
      - Pop a node.
      - Add it to ordering.
      - Reduce indegree of its neighbors.
      - If neighbor becomes 0 → push into queue.
4. If at end:
   - Processed nodes == total nodes → DAG
   - Else → cycle exists

#### Questions

> consider this DAG:
> 
> 1 → 3
> 
> 2 → 3
> 
> 3 → 4
>
> Questions:
>
> How many possible topological orderings exist?
>
> Is the ordering unique?
>
> At which step does branching occur?

the topological ordering is - 1,2,3,4 
2,1,3,4

Topological ordering is not unique.
It has branching.

> Uniqueness condition:
>
> At every step of Kahn’s algorithm,
> there must be exactly one node with indegree 0.
>
> The moment you see more than one → multiple valid orderings.

#### Question

> 1 → 2
> 
> 1 → 3
> 
> 2 → 4
> 
> 3 → 4
>
> Questions:
>
> How many topological orderings exist?
>
> Is the order unique?
>
> At which step does branching occur?

there are two topological ordering existing. Order is not unique. 1,2,3,4 1,3,2,4 Branching occurs at time when there are two nodes 2,3 both have indegree = 0. Actually, second step once node 1 is processed. At first step, single node with indegree =0. Then, 2,3. Now, we've two paths, if we select 2 first, we get 1,2,3,4. Else, 1,3,2,4.


#### Linear Chain / Directed Path

> Suppose a DAG has:
>
> Exactly n nodes
>
> Exactly n - 1 edges
>
> And topological ordering is unique.
>
> What must be true about the structure of this graph?

n nodes and n-1 edges with topological ordering as unique => at each step, we're getting a single node with indegree = 0. No branching occurred. 

It means direction from a start node goes straight to end/last node. There exists a path. I don't recall the exact graph name but I can visualize it. 1 -> 2 -> 3 -> 4 -> 5 5 nodes and 4 edges

So, it's a linear chain or a directed path.

```
v1 → v2 → v3 → v4 → ... → vn
```

Given:
- n nodes
- n - 1 edges
- DAG
- Unique topological ordering

We deduced:
- No branching allowed.
- Exactly one node has indegree 0 at every step.
- Every node (except first) has indegree 1.
- Every node (except last) has outdegree 1.

A single straight chain.

#### DAG vs Linear Chain

**1️⃣ Does every DAG have n - 1 edges?**

No.

That condition came from the specific constraint in the previous problem.

A DAG can have:
- 0 edges
- 1 edge
- n - 1 edges
- Much more than n - 1 edges

**2️⃣ Does every DAG have unique topological ordering?**

Also ❌ No.

Most DAGs do NOT have unique ordering.

**3️⃣ When Is Topological Order Unique?**

> A DAG has a unique topological ordering
iff at every step exactly one node has indegree 0

This enforces:

> The DAG must be a directed path.

Directed path is a special DAG.


```
Directed Graph
    ├── DAG
    │     ├── DAG with unique topo order (directed path)
    │     └── General DAG (many valid topo orders)
    └── Cyclic Directed Graph
```