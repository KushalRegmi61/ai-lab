# Hand-Trace Backup (Paper/Viva Format Insurance)

Source: the 4 photographed PYQ slips, which show an older paper/viva exam pattern (3 questions per slip, no live coding): one Prolog CSP problem + one Search-or-Adaline problem + one GA sub-topic hand-trace. Keep this as insurance in case any part of your slip is paper-based rather than a coding question.

## Quick answers, condensed (full derivations are in the numbered topic files)

**A. Crypto-Arithmetic A+A=BC** → `A=5, B=1, C=0` (5+5=10). See `03_prolog_csp.md`.

**B. Map Colouring (Australia)** → `WA=red, NT=green, SA=blue, Q=red, NSW=green, V=red, T=`any. See `03_prolog_csp.md`.

**C. Water Jug (4L,3L), target 2L, BFS** → 4 moves: `(0,0)→(0,3)→(3,0)→(3,3)→(4,2)`. See `14_water_jug_bfs.md`.

**D. A* Arad→Bucharest** → `Arad-Sibiu-RimnicuVilcea-Pitesti-Bucharest`, cost 418. See `05_astar_greedy.md`.

**E. Adaline for AND (bipolar)** → converges in 1 epoch to `w1=0.529, w2=0.559, b=-0.5` (real lab result). Delta rule: `y_in=b+Σxiwi`; `wi(new)=wi(old)+α(t−y_in)xi`; `b(new)=b(old)+α(t−y_in)`. See `10_perceptron_adaline.md`.

**F. GA hand-traces:**
- Selection (roulette): fitness 8,5,2,1 (total 16) → probabilities 0.50,0.3125,0.125,0.0625 → cumulative bands → draw random r, pick the band.
- Crossover: `1011|110` × `0100|001` → children `1011001`, `0100110`.
- Mutation: `10000011`, rate 0.08, flip bit 5 → `10001011`.
See `09_genetic_algorithm.md`.

**G. 4-Queens Prolog** → `(A,B,C,D) = (2,4,1,3)` or `(3,1,4,2)`. See `03_prolog_csp.md`.

**H. "What does Sue like?" FOPL puzzle** → `X = train` (Sue likes whatever Ann plays with; Ann plays with the train). See `02_prolog_fopl.md` for the closely-related "criminal," "Steve," and "Horses/Charlie" variants that are the lab's actual canonical examples.

## Exam-day habits for a paper/viva slip
- State **Variables → Domain → Constraints** explicitly before writing any CSP code — matches Lab Sheet 5's own framing.
- Name the data structure for search questions: FIFO queue (BFS), stack (DFS), priority queue by `f(n)` (A*).
- For GA questions, always state *why* each phase exists (selection→fitter parents, crossover→combine traits, mutation→diversity) — a very common "why" follow-up.
- If you blank on exact syntax, say the logic in plain English first — partial credit is real and it buys you thinking time.
