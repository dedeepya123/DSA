# 0-1 BFS — Shortest Path in a Graph with Edge Weights 0 and 1

## 1. Objective

Given a graph with edge weights restricted to either `0` or `1`, find the shortest distance from a given source vertex to every other vertex.

The graph can be directed or undirected.

Unlike an unweighted graph, the shortest path is determined by the **sum of edge weights**, not the number of edges.

## 2. Identify Graph Properties

Before choosing an algorithm, identify:

1. Directed or undirected?
2. Connected or disconnected?
3. Weighted or unweighted?
4. If weighted, what values can the edge weights take?

For this problem:
- The graph can be directed or undirected.
- It can be connected or disconnected.
- Edge weights are restricted to `0` and `1`.

## 3. Thought Process — Why Not Ordinary BFS?

We know that BFS finds the shortest path in an unweighted graph.

But why?

BFS explores vertices in increasing order of the number of edges from the source.

When all edges have equal weight, the first time we discover a vertex, its shortest distance is finalized.

### What happens when edge weights differ?

Consider this directed graph:

```text
          1
      0 -----> 1
      |
    0 |
      v
      2 -----> 3
          1

      1 ------> 3
           1
```

Edges:

```python
edges = [
    (0, 1, 1),
    (0, 2, 0),
    (1, 2, 1),
    (2, 3, 1),
    (1, 3, 1)
]
```

Source = `0`.

If we use ordinary BFS:

1. Process vertex 0.
   - Discover vertex 1 with distance 1.
   - Discover vertex 2 with distance 0.
   - Queue: `[1, 2]`

2. Process vertex 1.
   - Vertex 3 is discovered with distance 2.
   - Queue: `[2, 3]`

3. Process vertex 2.
   - Vertex 3 can be reached with distance 1.
   - But ordinary BFS using a visited-on-discovery check would not update it.

The actual shortest distance to vertex 3 is 1, not 2.

**Conclusion:** Ordinary BFS cannot directly guarantee shortest paths when edge weights differ.

## 4. Deriving 0-1 BFS

We need to process vertices in increasing order of their current shortest distances.

What data structure can help?

Two possibilities:

- Min-heap: always process the vertex with the smallest known distance.
- Deque: exploit the fact that edge weights are restricted to 0 and 1.

A min-heap is used by Dijkstra's algorithm.

For 0-1 BFS, we can use a deque because there are only two possible edge weights.

### Key idea

When processing a vertex `curr`:

- If the edge weight is `0`, the new distance remains the same. Add the neighbor to the **front** of the deque.
- If the edge weight is `1`, the new distance increases by one. Add the neighbor to the **back** of the deque.

```python
if wt == 0:
    queue.appendleft(nei)
else:
    queue.append(nei)
```

This ensures that vertices reached through zero-cost edges receive priority over vertices reached through one-cost edges.

## 5. Important Difference — BFS vs 0-1 BFS

### Ordinary BFS

In an unweighted graph, all edges have equal weight.

The first time a vertex is discovered, its shortest distance is final.

We use:

```python
if distance[nei] == float('inf'):
```

We do not need to update the distance again.

### 0-1 BFS

In a graph with weights 0 and 1, a vertex may be discovered through different paths with different costs.

We must check whether the new path improves the current best distance.

We use:

```python
newDistance = distance[curr] + wt

if newDistance < distance[nei]:
    distance[nei] = newDistance
```

This process is called **distance relaxation**.

A vertex may be inserted into the deque more than once if a better distance is discovered.

The deque ordering ensures that non-stale entries are processed in non-decreasing distance order. An outdated entry can be skipped.

## 6. Algorithm

1. Build the adjacency list.
   - Store `(neighbor, weight)` for each edge.
   - For undirected graphs, add both directions.

2. Initialize the distance array:
   - Set every distance to infinity.
   - Set the source distance to 0.

3. Initialize the deque:
   - Store `(distance, node)`.
   - Initially, add `(0, src)`.

4. Process the deque:
   - Remove the front entry `(curr_dist, curr)`.
   - If `curr_dist` is not equal to the current best distance of `curr`, skip this outdated entry.
   - Otherwise, explore its neighbors.

5. Relax every edge:
   - Calculate `newDistance = curr_dist + wt`.
   - If `newDistance` is smaller than the existing distance:
     - Update the distance.
     - If weight is 0, add the neighbor to the front.
     - If weight is 1, add the neighbor to the back.

6. Return the distance array.

## 7. Python Implementation

```python
from collections import defaultdict, deque

def zeroOneBFS(n, edges, src, isDirected=False):

    # Build adjacency list
    adj = defaultdict(list)

    for u, v, wt in edges:
        adj[u].append((v, wt))

        if not isDirected:
            adj[v].append((u, wt))

    # Initialize distances
    distance = [float('inf')] * n
    distance[src] = 0

    # Store (distance, node)
    queue = deque([(0, src)])

    while queue:

        curr_dist, curr = queue.popleft()

        # Skip outdated entries
        if curr_dist != distance[curr]:
            continue

        for nei, wt in adj[curr]:

            newDistance = curr_dist + wt

            if newDistance < distance[nei]:

                distance[nei] = newDistance

                if wt == 0:
                    queue.appendleft((newDistance, nei))
                else:
                    queue.append((newDistance, nei))

    return distance
```

## 8. Why Store (Distance, Node) in the Deque?

We can store only the node and read its current distance from the distance array.

However, storing `(distance, node)` makes the implementation easier to reason about.

Consider this situation:

- Vertex `X` is first discovered with distance 5.
- Later, another path improves its distance to 3.
- There may still be an older deque entry `(5, X)`.

When that older entry is popped:

```python
if curr_dist != distance[curr]:
    continue
```

We skip it because the best-known distance is now 3.

This prevents unnecessary processing of outdated entries.

## 9. Dry Run

Consider:

```python
n = 4
edges = [
    (0, 1, 1),
    (0, 2, 0),
    (1, 2, 1),
    (2, 3, 1),
    (1, 3, 1)
]

src = 0
```

### Initial state

```text
distance = [0, inf, inf, inf]
deque = [(0, 0)]
```

### Processing

| Current vertex | Relaxation | Deque after processing | Distance |
|---|---|---|---|
| 0 | 1 gets distance 1; 2 gets distance 0 | [(0,2), (1,1)] | [0,1,0,inf] |
| 2 | 3 gets distance 1 | [(1,1), (1,3)] | [0,1,0,1] |
| 1 | No improvement to 2 or 3 | [(1,3)] | [0,1,0,1] |
| 3 | No outgoing edges | [] | [0,1,0,1] |

Final distances:

```python
[0, 1, 0, 1]
```

Notice that vertex 3 is reached through vertex 2 with total cost 1.

The deque ensures that the zero-weight transition from vertex 0 to vertex 2 gets processed before the one-weight transition to vertex 1.

## 10. Complexity Analysis

Let:
- `V` = number of vertices.
- `E` = number of edges.

### Time Complexity: O(V + E)

- Building the adjacency list: O(V + E).
- Each successful relaxation inserts an entry into the deque.
- In 0-1 BFS, the deque maintains the required distance ordering.
- With edge weights restricted to 0 and 1, the algorithm performs O(V + E) total work.

A useful interview distinction:

Unlike ordinary BFS, a vertex can be inserted more than once. Do not justify the complexity by claiming every vertex is enqueued exactly once.

### Space Complexity

| Data structure | Space |
|---|---:|
| Adjacency list | O(V + E) |
| Distance array | O(V) |
| Deque | O(V + E) upper bound |

- Auxiliary space excluding the graph: O(V + E) in the worst case.
- Total space including adjacency list: O(V + E).

## 11. Edge Cases

1. **Empty graph**
   - If `n = 0`, return an empty distance array when no source is supplied.

2. **Single vertex**
   - Source distance is 0.

3. **Disconnected graph**
   - Unreachable vertices retain infinity.

4. **Directed graph**
   - Follow only outgoing edges.

5. **Undirected graph**
   - Add both directions with the same weight.

6. **Zero-weight edges**
   - These do not increase the current distance.
   - Add their neighbors to the front of the deque.

7. **Multiple paths to the same vertex**
   - Relax the distance whenever a smaller cost is found.

8. **Zero-weight cycles**
   - The strict improvement condition prevents infinite repeated updates.

9. **Invalid edge weights**
   - The algorithm assumes every edge weight is either 0 or 1.

## 12. Comparison with Other Shortest-Path Algorithms

| Algorithm | Edge weights | Data structure | Time complexity |
|---|---|---|---|
| BFS | All equal (typically 1) | Queue | O(V + E) |
| 0-1 BFS | 0 or 1 | Deque | O(V + E) |
| Dijkstra | Non-negative | Min-heap | O((V + E) log V) with adjacency list and binary heap |
| Bellman-Ford | Can include negative weights | Repeated edge relaxation | O(VE) |

### When should we use which?

- All edges have equal weight → BFS.
- All edge weights are 0 or 1 → 0-1 BFS.
- Arbitrary non-negative edge weights → Dijkstra.
- Negative edge weights → Bellman-Ford, with negative-cycle considerations.

## 13. Quick Recall

- Ordinary BFS relies on equal edge weights.
- Different weights require distance relaxation.
- 0-1 BFS works only when weights are 0 or 1.
- Use a deque.
- Weight 0 → appendleft.
- Weight 1 → append.
- Update only when the new distance is smaller.
- Store `(distance, node)` to identify stale entries.
- Skip outdated entries.
- Time complexity: O(V + E).

## 14. Interview Questions

1. Why does ordinary BFS fail for different edge weights?
2. What is 0-1 BFS?
3. Why do we use a deque instead of a normal queue?
4. Why do zero-weight edges go to the front?
5. Why do one-weight edges go to the back?
6. What is distance relaxation?
7. Why can a vertex be inserted into the deque multiple times?
8. Why do we check for outdated entries?
9. Why is the first discovery not necessarily final?
10. What happens if an edge has weight 2?
11. How is 0-1 BFS different from Dijkstra's algorithm?
