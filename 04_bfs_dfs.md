# BFS and DFS — Graph Search (Python)

Source: this session's teaching pass, code verified by execution. Confirmed exam topic (named directly by CR).

## The core idea
Both explore a graph for a path from start to goal. The only difference is the data structure holding "what to look at next":
- **BFS**: **queue** (FIFO). Explores level by level → finds the path with the fewest edges.
- **DFS**: **stack** (LIFO) or recursion. Dives deep down one branch first → less memory, no shortest-path guarantee.

## Tested code
```python
from collections import deque

graph = {
    'A': ['B', 'C'], 'B': ['A', 'D', 'E'], 'C': ['A', 'F'],
    'D': ['B'], 'E': ['B', 'F'], 'F': ['C', 'E'],
}

def bfs(graph, start, goal):
    visited = {start}
    queue = deque([[start]])
    while queue:
        path = queue.popleft()        # FIFO: oldest path first
        node = path[-1]
        if node == goal:
            return path
        for nbr in graph[node]:
            if nbr not in visited:
                visited.add(nbr)
                queue.append(path + [nbr])
    return None

def dfs(graph, start, goal, visited=None, path=None):
    if visited is None:
        visited, path = set(), [start]
    visited.add(start)
    if start == goal:
        return path
    for nbr in graph[start]:
        if nbr not in visited:
            result = dfs(graph, nbr, goal, visited, path + [nbr])
            if result:
                return result
    return None

print("BFS A->F:", bfs(graph, 'A', 'F'))   # ['A', 'C', 'F']
print("DFS A->F:", dfs(graph, 'A', 'F'))   # ['A', 'B', 'E', 'F']
```

## The one line that IS the whole difference
```python
path = queue.popleft()   # BFS
# vs (conceptually) path = stack.pop()  # DFS -- LIFO instead of FIFO
```
DFS via recursion works because the function call stack does the LIFO bookkeeping automatically.

## Properties table (common follow-up)
| | Complete? | Optimal? | Memory |
|---|---|---|---|
| BFS | Yes (finite branching) | Yes, IF all edges cost equally | High — stores whole frontier, O(b^d) |
| DFS | Only with visited-tracking | No | Low — O(depth) |

## Say out loud
- "Why is BFS optimal but DFS isn't?" → BFS explores all length-1 paths, then all length-2, etc, so the first time it reaches the goal it's via the fewest edges. DFS can wander down a long branch first.
- "Which uses less memory and why?" → DFS, because it only needs the current path, not the whole frontier.
