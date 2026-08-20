# A* Search and Greedy Best-First Search (Python)

Source: this session's teaching pass + Lab Report 4 (Search Strategies), cross-verified. Confirmed exam topic.

## The core idea
Both are **informed** search — use a heuristic `h(n)` estimating distance from node n to goal.
- **Greedy Best-First**: expand smallest `h(n)`. Fast, not optimal (ignores cost already paid).
- **A\***: expand smallest `f(n) = g(n) + h(n)`. Optimal if `h(n)` never overestimates (admissible).

## Worked example — Arad to Bucharest (classic AIMA Romania map)
Straight-line distances to Bucharest (the heuristic): Arad=366, Sibiu=253, Fagaras=178, RimnicuVilcea=193, Pitesti=100, Bucharest=0.

**A\* trace:**
```
Arad -> Sibiu (g=140,h=253,f=393)
Sibiu -> RimnicuVilcea (g=220,h=193,f=413)   [beats Fagaras: g=239,h=178,f=417]
RimnicuVilcea -> Pitesti (g=317,h=100,f=417)   [ties Fagaras f=417; either order still ends at 418]
Pitesti -> Bucharest (g=418,h=0,f=418)  -- goal, lowest f on frontier -> STOP
```
**Path: Arad → Sibiu → RimnicuVilcea → Pitesti → Bucharest, cost = 418.**

**Greedy trace:** at Sibiu, compares only h: h(Fagaras)=178 < h(RimnicuVilcea)=193 → picks Fagaras.
**Path: Arad → Sibiu → Fagaras → Bucharest, cost = 450** — worse overall, even though each step "looked" closer.

**Say out loud:** "A* considers cost already paid (g), Greedy doesn't — that's the whole difference. Lab Report 4's exhaustive BFS/UCS/DFS/IDDFS/Greedy/A* comparison table confirms only UCS and A* find the optimal cost-418 path; the rest find the shorter-hop but costlier 450 path."

## Tested code (full exam map — verified, prints exactly what is shown)

Type each road **once** in `edges`; the loop writes it into both cities. Roads are two-way — if
Arad lists Sibiu but Sibiu does not list Arad, A* can drive in but never drive out. Building it
this way makes that mistake impossible.

```python
import heapq

edges = [
    ('Arad','Zerind',75),      ('Arad','Sibiu',140),     ('Arad','Timisoara',118),
    ('Zerind','Oradea',71),    ('Oradea','Sibiu',151),
    ('Timisoara','Lugoj',111), ('Lugoj','Mehadia',70),   ('Mehadia','Dobreta',75),
    ('Dobreta','Craiova',120), ('Craiova','RimnicuVilcea',146), ('Craiova','Pitesti',138),
    ('Sibiu','Fagaras',99),    ('Sibiu','RimnicuVilcea',80),
    ('RimnicuVilcea','Pitesti',97),
    ('Fagaras','Bucharest',211), ('Pitesti','Bucharest',101),
    ('Bucharest','Giurgiu',90),  ('Bucharest','Urziceni',85),
    ('Urziceni','Hirsova',98),   ('Hirsova','Eforie',86),
    ('Urziceni','Vaslui',142),   ('Vaslui','Iasi',92),   ('Iasi','Neamt',87),
]

graph = {}
for a, b, c in edges:
    graph.setdefault(a, {})[b] = c
    graph.setdefault(b, {})[a] = c

print(graph)

# straight-line distance to Bucharest, copied from the question paper
h = {'Arad':366,'Bucharest':0,'Craiova':160,'Dobreta':242,'Eforie':161,
     'Fagaras':178,'Giurgiu':77,'Hirsova':151,'Iasi':226,'Lugoj':244,
     'Mehadia':241,'Neamt':234,'Oradea':380,'Pitesti':100,'RimnicuVilcea':193,
     'Sibiu':253,'Timisoara':329,'Urziceni':80,'Vaslui':199,'Zerind':374}

def path_cost(graph, path):
    return sum(graph[path[i]][path[i+1]] for i in range(len(path)-1))

def a_star(graph, start, goal, h):
    pq = [(h[start], 0, start, [start])]      # (f, g, node, path)
    visited = set()
    while pq:
        f, g, node, path = heapq.heappop(pq)
        if node in visited: continue
        visited.add(node)
        if node == goal:
            return path, g
        for nbr, cost in graph[node].items():
            if nbr not in visited:
                new_g = g + cost
                heapq.heappush(pq, (new_g + h[nbr], new_g, nbr, path + [nbr]))
    return None, float('inf')

def greedy_best_first(graph, start, goal, h):
    pq = [(h[start], start, [start])]         # note: no g at all
    visited = set()
    while pq:
        hval, node, path = heapq.heappop(pq)
        if node in visited: continue
        visited.add(node)
        if node == goal:
            return path, path_cost(graph, path)   # cost computed after, not used to steer
        for nbr in graph[node]:
            if nbr not in visited:
                heapq.heappush(pq, (h[nbr], nbr, path + [nbr]))
    return None, float('inf')

print(a_star(graph, 'Arad', 'Bucharest', h))
# (['Arad', 'Sibiu', 'RimnicuVilcea', 'Pitesti', 'Bucharest'], 418)
print(greedy_best_first(graph, 'Arad', 'Bucharest', h))
# (['Arad', 'Sibiu', 'Fagaras', 'Bucharest'], 450)
```

## Two mistakes that cost marks
1. **Missing `h` entry** → `KeyError` on the first expansion. Every city named in `edges` must
   appear in `h`. Check with one line before you run: `print(set(graph) - set(h))` — must be empty.
2. **One-way roads** → A* silently returns a wrong path or nothing. Always build both directions.

If short on time, you may drop Giurgiu/Urziceni/Hirsova/Eforie/Vaslui/Iasi/Neamt (they are past
Bucharest and never expanded) — but drop them from `h` too. Answer is still 418.

## `heapq`, explained
Python's built-in priority queue. `heappop` always returns the smallest item. Tuples compare element-by-element, so `(f, g, node, path)` always pops lowest `f` first.

## Admissible heuristic
`h(n)` must never overestimate true remaining cost. Straight-line distance is admissible for road maps (roads bend, the straight line never exceeds actual road distance). "Why is A* optimal?" → because an admissible heuristic never causes A* to prematurely abandon what could still be the cheapest path.
