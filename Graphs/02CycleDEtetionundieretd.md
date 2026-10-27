# Undirected Cycle Detection using BFS

## 1. Objective

Given an **undirected graph** with `n` vertices and an edge list, determine whether the graph contains a cycle.

Return:
- `True` if at least one cycle exists.
- `False` otherwise.

## 2. Understand the graph first

Before choosing an algorithm, identify the graph properties:

- **Directed or undirected?** This approach is for undirected graphs.
- **Connected or disconnected?** The graph may contain multiple components.
- **Representation:** An edge list is given, so build an adjacency list.
- **Vertices:** Assume the vertices are numbered from `0` to `n - 1`, including isolated vertices that do not appear in any edge.

For an undirected edge `(u, v)`, add both directions:

```python
adjlist[u].append(v)
adjlist[v].append(u)
```

## 3. Core intuition: Why track the parent?

During BFS, we visit a vertex and examine its neighbors.

If a neighbor is unvisited, we discover it and add it to the queue.

But if a neighbor is already visited, **does that automatically mean a cycle exists?**

No. In an undirected graph, every edge can be traversed in both directions. When we visit a vertex, its parent is already visited.

For example:

```text
0 ---- 1 ---- 2
```

When processing vertex `1`, we see vertex `0` again. This is not a cycle; `0` is simply the parent through which we reached `1`.

Therefore, the rule is:

- **Unvisited neighbor:** Mark it visited and enqueue it, recording the current vertex as its parent.
- **Visited neighbor that is the parent:** Ignore it.
- **Visited neighbor that is not the parent:** A cycle exists.

The key condition is:

```python
elif nei != parent:
    return True
```

This reasoning assumes a simple undirected graph without parallel edges. Multigraphs require tracking the parent edge rather than just the parent vertex.

## 4. Algorithm

### Step 1: Build the adjacency list

Store every undirected edge in both directions.

### Step 2: Maintain a shared visited set

Create one `visited` set for the entire graph. It must be shared across BFS calls so that vertices already explored in previous components are not processed again.

### Step 3: Run BFS with parent tracking

For each BFS call:

1. Initialize the queue with `(start_node, -1)`, where `-1` means there is no parent for the starting node.
2. Mark the starting node as visited.
3. Remove `(current, parent)` from the queue.
4. Examine each neighbor:
   - If unvisited, mark it visited and enqueue `(neighbor, current)`.
   - If visited and not equal to `parent`, return `True`.
5. If BFS finishes without detecting a cycle, return `False`.

### Step 4: Handle disconnected graphs

Loop through every vertex from `0` to `n - 1`.

Whenever a vertex is unvisited, start BFS from it. This ensures that every connected component, including isolated vertices, is checked.

If any BFS detects a cycle, return `True`. If all components are explored without finding one, return `False`.

## 5. Implementation

```python
from collections import defaultdict, deque


def has_cycles(edges, n):
    # Build the adjacency list.
    adjlist = defaultdict(list)

    for u, v in edges:
        adjlist[u].append(v)
        adjlist[v].append(u)

    def bfs(node, visited):
        queue = deque([(node, -1)])
        visited.add(node)

        while queue:
            current, parent = queue.popleft()

            for nei in adjlist[current]:
                if nei not in visited:
                    visited.add(nei)
                    queue.append((nei, current))

                elif nei != parent:
                    return True

        return False

    visited = set()

    # Check every connected component.
    for node in range(n):
        if node not in visited:
            if bfs(node, visited):
                return True

    return False
```

## 6. Dry run: A graph containing a cycle

Consider:

```text
    0
   / \
  1---2
```

Edges: `[(0, 1), (1, 2), (2, 0)]`

Start BFS from vertex `0`.

| Step | Current vertex | Observation | Result |
|---|---|---|---|
| 1 | `0` | Neighbors `1` and `2` are unvisited | Enqueue both with parent `0` |
| 2 | `1` | Neighbor `0` is its parent | Ignore |
| 3 | `1` | Neighbor `2` is already visited and is not parent `0` | Cycle detected |

The graph contains a cycle because there is an edge between two already discovered vertices that are not in a parent-child relationship.

## 7. Complexity analysis

Let `V` be the number of vertices and `E` the number of edges.

- **Building the adjacency list:** `O(V + E)` including initialization/representation of the vertices.
- **BFS across all components:** `O(V + E)`, because each vertex is processed once and each adjacency-list entry is examined once.
- **Total time complexity:** `O(V + E)`.
- **Auxiliary traversal space:** `O(V)` for `visited` and the queue.
- **Total space including adjacency list:** `O(V + E)`.

## 8. Edge cases to test

| Case | Expected result | Reason |
|---|---|---|
| `n = 0`, no edges | `False` | No vertices or cycles |
| One vertex, no edges | `False` | An isolated vertex has no cycle |
| `[(0, 1), (1, 2)]`, `n = 3` | `False` | A simple path |
| `[(0, 1), (1, 2), (2, 0)]`, `n = 3` | `True` | Triangle cycle |
| `[(0, 1), (2, 3)]`, `n = 4` | `False` | Disconnected components, neither contains a cycle |
| `[(0, 1), (1, 2), (2, 3), (3, 0)]`, `n = 4` | `True` | A four-vertex cycle |
| `n = 4`, no edges | `False` | Four isolated vertices |

**Important:** If self-loops or parallel edges are permitted, clarify the graph's cycle definition and adjust the implementation accordingly.

## 9. Pattern to remember

**Undirected cycle detection = BFS/DFS + parent tracking.**

The parent check distinguishes the ordinary reverse traversal of an undirected edge from a genuine cycle.

For a disconnected graph, run the traversal from every unvisited vertex.

### Quick recall questions

1. Why is a visited neighbor alone insufficient to detect a cycle in an undirected graph?
2. Why do we enqueue `(neighbor, current)` rather than just `neighbor`?
3. Why must `visited` be shared across components?
4. Why do we start BFS from every unvisited vertex?
5. How would this reasoning differ for directed cycle detection?

