# Genetic Algorithm (Python)

Source: this session's teaching pass (toy example, code verified) + Lab Sheet 6 / Lab Report 6 (the real, much larger assignment). Confirmed exam topic (implementation, not just hand-trace).

## The 5 phases (name these explicitly if asked "what is a GA")
1. **Initial population** — random candidate solutions ("chromosomes").
2. **Fitness function** — score for how good each candidate is.
3. **Selection** — keep/pick fitter individuals as parents.
4. **Crossover** — combine two parents into new children.
5. **Mutation** — small random changes for diversity, avoids premature convergence.
Repeat 2–5 each generation until good enough or a generation cap.

## Exam-viable toy example — evolve random letters into "HELLO"
(The lab's *actual* Lab 6 assignment is a much bigger painter-robot simulation — see below. This toy version demonstrates the identical 5-phase algorithm in a form you can actually type and run live in a short exam slot.)
```python
import random
random.seed(1)

TARGET = "HELLO"
GENES = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
POP_SIZE = 20

def create_chromosome():
    return [random.choice(GENES) for _ in range(len(TARGET))]

def fitness(chromo):
    return sum(1 for g, t in zip(chromo, TARGET) if g == t)

def selection(pop):
    return sorted(pop, key=fitness, reverse=True)[:POP_SIZE // 2]   # keep fittest half

def crossover(p1, p2):
    point = random.randint(1, len(TARGET) - 1)
    return p1[:point] + p2[point:], p2[:point] + p1[point:]

def mutate(chromo, rate=0.1):
    return [random.choice(GENES) if random.random() < rate else g for g in chromo]

population = [create_chromosome() for _ in range(POP_SIZE)]
generation = 0
while True:
    population = sorted(population, key=fitness, reverse=True)
    best = population[0]
    if fitness(best) == len(TARGET) or generation > 500:
        break
    parents = selection(population)
    next_gen = parents.copy()
    while len(next_gen) < POP_SIZE:
        p1, p2 = random.sample(parents, 2)
        c1, c2 = crossover(p1, p2)
        next_gen += [mutate(c1), mutate(c2)]
    population = next_gen[:POP_SIZE]
    generation += 1

print(f"Gen {generation}: {''.join(best)}  (fitness {fitness(best)}/{len(TARGET)})")
# Gen 45: HELLO  (fitness 5/5)
```

## Mapping code to phases
| Function | Phase |
|---|---|
| `create_chromosome()` | 1. Initial population |
| `fitness(chromo)` | 2. Fitness |
| `selection(pop)` | 3. Selection (truncation — keep top half) |
| `crossover(p1,p2)` | 4. Crossover (single-point) |
| `mutate(chromo)` | 5. Mutation (per-gene random-flip chance) |

## The REAL Lab 6 assignment (know this exists, in case asked about "the lab's GA problem" specifically)
A **painter robot** navigates a 20×40 room, chromosome = 54 genes (one action per local state `[current, forward, left, right]`, each state ∈ a small enum, `2×3×3×3=54` combinations). Genes: 0=no turn, 1=left, 2=right, 3=random. GA config used: population 50, **tournament selection (size 3) with 2 elites** (not truncation), single-point crossover, **mutation rate 0.002 per locus**, 200 generations. Result: 99%+ coverage efficiency in an empty room; a policy evolved on an empty room transfers poorly to a furnished room (94.6% mean, one run as low as 0.14%) versus a policy evolved specifically for the furnished room (98.9% mean) — demonstrating that GA policies are specialized to the environment they were evolved in.
**Too large to reproduce live in an exam** — if asked, describe the setup and the 5-phase mapping; use the toy code above as your actual implementation.

## Hand-trace versions (paper-based backup)
- **Selection (roulette wheel):** fitness 8,5,2,1 (total=16) → probabilities 0.50,0.3125,0.125,0.0625 → cumulative 0.50,0.8125,0.9375,1.0 → draw random r∈[0,1], pick the band.
- **Crossover:** P1=`1011|110`, P2=`0100|001`, cut after bit 4 → C1=`1011001`, C2=`0100110`.
- **Mutation:** child `10000011`, rate 0.08, flip bit 5 → `10001011`.

## Say out loud
"Selection favors fitter parents; crossover combines good traits; mutation preserves diversity and avoids getting stuck in a local optimum." Examiners often ask "why" for each phase — have this one-liner ready.
