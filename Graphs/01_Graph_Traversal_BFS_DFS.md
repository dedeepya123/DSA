# Graph Traversal — BFS and DFS

## 1. Problem Understanding

Given a graph \(G = (V, E)\), we want to traverse its vertices by following the edges.

**Objective:** Visit the vertices reachable from a given source, or visit every vertex in the graph if a complete traversal is required.

Before choosing an approach, identify the graph properties:

- **Direction:** Directed or undirected?
- **Connectivity:** Connected or disconnected?
- **Scope:** Traversal from a source or traversal of the entire graph?

### Choosing the Algorithm

- **BFS (Breadth-First Search):** Explores the graph level by level.
- **DFS (Depth-First Search):** Explores as deeply as possible before backtracking.

Both algorithms can be used for directed and undirected graphs.

**Important:** Starting from a source does not guarantee that every vertex will be visited. We visit only the vertices reachable from that source.

---

## 2. Breadth-First Search (BFS)

### Intuition

BFS explores vertices in increasing order of their distance in terms of the number of edges from the source.

We use a **queue (FIFO)** because vertices discovered earlier must be processed earlier.

### Algorithm

1. Build the adjacency list.
2. Identify the source vertex.
3. Initialize a queue and a visited structure.
4. Mark the source as visited and add it to the queue.
5. While the queue is not empty:
   - Remove the front vertex from the queue.
   - Process the vertex.
   - For every neighbor:
     - If the neighbor is unvisited:
       - Mark it as visited.
       - Add it to the queue.
6. Stop when the queue becomes empty.

### Why mark a vertex when enqueueing it?

A vertex may be reachable through multiple neighbors. Marking it visited as soon as it is added to the queue prevents it from being enqueued multiple times.

### Complexity

Assuming an adjacency-list representation:

- **Time:** \(O(V + E)\)
- **Auxiliary space:** \(O(V)\)

Each vertex is processed once, and each adjacency-list entry is examined once.

---

## 3. Depth-First Search (DFS)

### Intuition

DFS explores one path as deeply as possible before returning to explore other paths.

We can implement DFS recursively using the **call stack**, or iteratively using an explicit stack.

### Recursive Algorithm

1. Mark the current vertex as visited.
2. Process the vertex.
3. For every neighbor:
   - If the neighbor is unvisited, recursively explore it.
4. When no unvisited neighbors remain, return to the previous call.

This return to a previous call is called **backtracking**.

### Complexity

Assuming an adjacency-list representation:

- **Time:** \(O(V + E)\)
- **Auxiliary space:** \(O(V)\) in the worst case, including the visited structure and recursion stack.

For very deep graphs, iterative DFS can avoid Python's recursion-depth limitations.

---

## 4. Connected vs. Disconnected Graphs

The traversal strategy depends on whether we need to explore only the vertices reachable from a source or the entire graph.

| Graph type | Traversal from one source | Traversal of the entire graph |
|---|---|---|
| Connected, undirected | Visits every vertex | One BFS/DFS traversal |
| Disconnected, undirected | Visits only the source's component | Restart from every unvisited vertex |
| Directed | Visits only vertices reachable from the source | Restart from every unvisited vertex to visit all vertices |

In a directed graph, reachability depends on edge direction. Even if the graph is connected when edge directions are ignored, a particular source might not reach every vertex.

### Traversing the Entire Graph

To visit every vertex, iterate through all vertices. Whenever an unvisited vertex is found, start a new BFS or DFS from it.

- In an undirected graph, this discovers the connected components.
- In a directed graph, this visits all vertices, but the resulting traversal trees are **not necessarily strongly connected components**.

---

## 5. BFS vs. DFS

| Property | BFS | DFS |
|---|---|---|
| Data structure | Queue | Recursion stack or explicit stack |
| Exploration | Level by level | Deep first, then backtrack |
| Shortest path | Finds shortest paths in unweighted graphs | Does not generally guarantee shortest paths |
| Connected components | Yes, in undirected graphs | Yes, in undirected graphs |
| Cycle detection | Possible with appropriate tracking | Possible with appropriate tracking |
| Time complexity | \(O(V + E)\) | \(O(V + E)\) |

The traversal order may differ depending on the order of neighbors in the adjacency list.

---

## 6. Mental Model to Remember

**Graph traversal = exploration + visited tracking.**

- BFS → Queue → Level-by-level exploration.
- DFS → Recursion/Stack → Explore deeply, then backtrack.
- Visited structure → Prevents repeated processing and infinite loops.
- Source traversal → Visits only reachable vertices.
- Full-graph traversal → Restart from every unvisited vertex.
- Adjacency list → \(O(V + E)\) time complexity.

### Problem-Solving Checklist

When given a graph problem, ask:

1. What exactly must be visited or discovered?
2. Is the graph directed or undirected?
3. Do I need to start from a source or cover the entire graph?
4. Can I use BFS or DFS?
5. Do I need additional tracking for components, distances, or cycles?

**Key distinction:** Traversal is the mechanism. Connectivity, shortest paths, and cycle detection are separate problems that can use traversal as a building block.

---

## 7. Practice to Lock the Pattern

1. Implement BFS traversal from a given source.
2. Implement DFS traversal from a given source.
3. Count connected components in a disconnected undirected graph.

The goal is to derive the approach and implement it independently, not memorize the code.
