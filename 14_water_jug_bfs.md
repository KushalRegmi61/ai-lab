# Water Jug Problem — BFS State-Space Search (Python)

Source: this session's teaching pass, code verified by execution. Appeared in 2 of 4 historical PYQ slips ("Solve water jug using BFS").

## Turning it into a search problem
| Search concept | Water jug meaning |
|---|---|
| State | `(x, y)` = current litres in jug1, jug2 |
| Start state | `(0, 0)` |
| Goal test | `x == target` or `y == target` |
| Operators | fill jug1, fill jug2, empty jug1, empty jug2, pour jug1→jug2, pour jug2→jug1 |

## Tested code
```python
from collections import deque

def water_jug_bfs(cap1, cap2, target):
    start = (0, 0)
    visited = {start}
    queue = deque([[start]])
    while queue:
        path = queue.popleft()
        x, y = path[-1]
        if x == target or y == target:
            return path
        d1 = min(x, cap2 - y)   # pour jug1 -> jug2
        d2 = min(y, cap1 - x)   # pour jug2 -> jug1
        next_states = [
            (cap1, y), (x, cap2),          # fill jug1, fill jug2
            (0, y), (x, 0),                # empty jug1, empty jug2
            (x - d1, y + d1),              # pour jug1 -> jug2
            (x + d2, y - d2),              # pour jug2 -> jug1
        ]
        for state in next_states:
            if state not in visited:
                visited.add(state)
                queue.append(path + [state])
    return None

path = water_jug_bfs(4, 3, 2)
for state in path: print(state)
print("Total moves:", len(path)-1)
# (0,0) (0,3) (3,0) (3,3) (4,2)
# Total moves: 4   -- provably minimal (verified: no 2 exists at levels 1-3)
```

## Verified minimal solution (4L, 3L jugs, target 2 litres)
```
Level 0: (0,0)                                start
Level 1: (4,0), (0,3)                          no 2 yet
Level 2: (4,3), (1,3), (3,0)                   no 2 yet
Level 3: (1,0), (3,3)                          no 2 yet
Level 4: (0,1), (4,2)  <-- jug2=2! GOAL
```
Path: fill jug2 `(0,3)` → pour jug2→jug1 `(3,0)` → fill jug2 again `(3,3)` → pour jug2→jug1 (jug1 can only take 1 more before overflow) `(4,2)` — jug2 now has exactly 2 litres. **4 moves, provably minimal** (checked no 2 appears at levels 1–3).

## Pour amount formula, explained
`d1 = min(x, cap2 - y)`: can't pour more than what's *in* the source (`x`), and can't pour more than *fits* in the destination (`cap2 - y`). Whichever is smaller is what actually moves.

## Say out loud
"The state space itself IS the graph — nodes are `(x,y)` pairs, edges are the 6 jug operations, and BFS explores it exactly like graph BFS: `deque`, `popleft`, `visited` set, build up a `path` list." The only new part vs plain graph BFS is that neighbors are *computed by formula* instead of looked up in a dictionary.
