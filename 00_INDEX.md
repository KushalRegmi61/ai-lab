# AI Lab Exam — Study Notes Index

Built from a full read of all 8 Lab Sheets, all 8 Lab Reports, and 4 photographed past-exam slips, plus the class CR's confirmation of this year's actual exam structure.

## Exam format (confirmed by CR)
- **5 question sets exist, you solve 3.** Seating is pre-planned by roll number; which sets you draw is the professor's (Basanta sir's) call — effectively random.
- **Confirmed topics, in Python:** Naive Bayes, Logistic Regression, BFS/DFS, Minimax, Alpha-Beta pruning, Neural Networks (Perceptron/Adaline, Backpropagation).
- **Also to prepare:** N-Queens, Tower of Hanoi, Genetic Algorithm implementation, A*, Greedy search.
- **Prolog part:** knowledge representation (facts/rules/FOPL) and basic proving via a goal query.
- **Logistics:** lab reports do not need to be shown/checked on exam day.
- The 4 historical PYQ slips (paper/viva style) show a *different, older* exam pattern: 3 questions per slip drawn from Prolog-CSP, Search/Adaline, and GA sub-topics. Kept as backup in case any part is still paper-based.

## Files in this folder

| File | Covers |
|---|---|
| `01_prolog_basics.md` | Facts, rules, atoms/variables, lists, HCF, family relations, list-processing (sum/length/append/filter/delete) |
| `02_prolog_fopl.md` | FOPL→Prolog translation, the "criminal" example, Steve/courses, Horses/Charlie, Student Evaluation forward+backward chaining, Monkey-Banana |
| `03_prolog_csp.md` | Crypto-arithmetic (classic + modern `clpfd`), N-Queens in Prolog, Map Colouring in Prolog (classic + `clpfd`) |
| `04_bfs_dfs.md` | BFS/DFS graph search in Python |
| `05_astar_greedy.md` | A* and Greedy Best-First search in Python, Romania map |
| `06_minimax_alphabeta.md` | Minimax and Alpha-Beta pruning in Python |
| `07_nqueens_python.md` | N-Queens backtracking in Python |
| `08_tower_of_hanoi.md` | Tower of Hanoi recursion in Python |
| `09_genetic_algorithm.md` | GA 5-phase theory, toy Python implementation, note on the real (painter-robot) lab assignment |
| `10_perceptron_adaline.md` | **Corrected**: Adaline delta-rule learning (unipolar/bipolar), McCulloch-Pitts fixed-weight gates, real lab results |
| `11_backpropagation_xor.md` | **Corrected**: MLP backprop for XOR, hand-built McCulloch-Pitts XOR net, real lab results |
| `12_naive_bayes_logistic_regression.md` | **Corrected**: full 3-way split pipeline, generative vs discriminative, decision-tree capacity/bias-variance, cross-entropy, MLE vs MAP |
| `13_agent_types.md` | Simple Reflex vs Model-Based agents (Vacuum World) — in the lab materials, not named by the CR, lower priority |
| `14_water_jug_bfs.md` | Water Jug problem as a BFS state-space search |
| `15_hand_trace_backup.md` | Paper/viva-style hand traces (crypto-arithmetic, A*, Adaline, GA) matching the older PYQ slip format |

## Priority order for tonight/today (given time is short)
1. Prolog (`01`–`03`) — strongest historical exam evidence, appeared in 4/4 past slips.
2. BFS/DFS, A*/Greedy, Minimax/Alpha-Beta (`04`–`06`) — explicitly named by the CR, least prior lab-sheet exposure.
3. N-Queens, Tower of Hanoi, GA (`07`–`09`) — quick wins, mostly reused logic.
4. Perceptron/Adaline, Backprop, Naive Bayes/LogReg (`10`–`12`) — longest code, but very formulaic once the pattern is seen.
5. `13`–`15` — read only if time remains.

## Corrections made in this pass (vs. earlier draft material)
- **Adaline ≠ classic Perceptron.** Adaline's delta-rule error uses the *raw* net input `y_in`, not the thresholded prediction. Fixed in `10_perceptron_adaline.md`.
- **Crypto-arithmetic**: the lab's real example is `SEND+MORE=MONEY` (with carries C1–C4), not just `A+A=BC`. Both are covered in `03_prolog_csp.md`.
- **Modern Prolog CSP style**: Lab 5's actual report uses SWI-Prolog's `library(clpfd)` (`ins`, `all_different`, `#=`, `label`), which is cleaner than the classical `member`+backtracking style from the lab sheet. Both shown in `03_prolog_csp.md` — use whichever your exam machine's Prolog supports.
- **Naive Bayes/Logistic Regression**: the real pipeline uses a 3-way stratified split (60/20/20 train/val/test) and includes a Decision Tree capacity study — not just a simple 2-way split. Fixed in `12_naive_bayes_logistic_regression.md`.
- **Horses/Charlie FOPL puzzle**: the literal facts given in Lab Sheet 2 do **not** state Bluebeard is a horse — so strictly, "is Charlie a horse?" is *not provable* from the given facts alone. The submitted lab report adds `horse(bluebeard).` as an extra fact to make it provable. Both readings are explained in `02_prolog_fopl.md` — know the subtlety in case it's asked as a trick "why" follow-up.
