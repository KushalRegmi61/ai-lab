# Prolog — Constraint Satisfaction Problems (CSP)

Source: Lab Sheet 5, Lab Report 5, plus PYQ slips (A+A=BC, 4-Queens, map coloring all appeared in actual past exams).

## The CSP framework — say this explicitly for any CSP question
> "A CSP has **Variables**, a **Domain** of possible values for each, and **Constraints** restricting valid combinations. A solution assigns every variable a value from its domain satisfying every constraint."

Two Prolog styles exist for this course. **Check which Prolog your exam machine runs** — old-style Turbo/Visual Prolog needs the classical style; modern SWI-Prolog can use either, but `clpfd` is cleaner if available.

---

## Style A — Classical generate-and-test (works on any Prolog dialect)
Generate every candidate with `member`, then filter with constraints (`<>`/`\=` = not equal, `=:=` = numeric equality).
```prolog
member(X,[X|_]).
member(X,[_|T]):- member(X,T).
```

### Crypto-arithmetic: A + A = BC (simple version, appeared on an actual PYQ slip)
```prolog
solution(A,B,C):-
    member(A,[0,1,2,3,4,5,6,7,8,9]),
    member(B,[1,2,3,4,5,6,7,8,9]),   % B<>0, BC is 2-digit
    member(C,[0,1,2,3,4,5,6,7,8,9]),
    A<>B, B<>C, A<>C,
    A+A =:= 10*B+C.
GOAL: solution(A,B,C).
```
By hand: 2A=10B+C, try A=5 → 2A=10 → B=1,C=0. **A=5,B=1,C=0** (5+5=10).

### Crypto-arithmetic: SEND + MORE = MONEY (the lab's actual worked example, uses carries)
Column equations (right to left), with carries C1..C4 ∈ {0,1}:
```
D+E = Y+10*C1
N+R+C1 = E+10*C2
E+O+C2 = N+10*C3
S+M+C3 = O+10*C4
M = C4
```
```prolog
solution([S,E,N,D,M,O,R,Y]):-
    C4=1,                              % M must be 1 (else no carry needed at all)
    member(C1,[0,1]), member(C2,[0,1]), member(C3,[0,1]),
    member(E,[0,1,2,3,4,5,6,7,8,9]), member(N,[0,1,2,3,4,5,6,7,8,9]),
    member(D,[0,1,2,3,4,5,6,7,8,9]), member(M,[0,1,2,3,4,5,6,7,8,9]),
    member(O,[0,1,2,3,4,5,6,7,8,9]), member(R,[0,1,2,3,4,5,6,7,8,9]),
    member(Y,[0,1,2,3,4,5,6,7,8,9]), member(S,[0,1,2,3,4,5,6,7,8,9]),
    S<>E, S<>N, S<>D, S<>M, S<>O, S<>R, S<>Y,
    E<>N, E<>D, E<>M, E<>O, E<>R, E<>Y,
    N<>D, N<>M, N<>O, N<>R, N<>Y,
    D<>M, D<>O, D<>R, D<>Y,
    M<>O, M<>R, M<>Y,
    O<>R, O<>Y,
    R<>Y,
    D+E=Y+10*C1, N+R+C1=E+10*C2, E+O+C2=N+10*C3, S+M+C3=O+10*C4, M=C4.
GOAL: solution([S,E,N,D,M,O,R,Y]).
```
**Actual verified answer (from the lab report): S=9,E=5,N=6,D=7,M=1,O=0,R=8,Y=2** → 9567+1085=10652.

### N-Queens (4-Queens shown; generalizes to 8)
```prolog
noattack(X1,X2,D):- X1<>X2, abs(X1-X2)<>D.
solution(A,B,C,D):-
    member(A,[1,2,3,4]), member(B,[1,2,3,4]),
    member(C,[1,2,3,4]), member(D,[1,2,3,4]),
    noattack(A,B,1), noattack(A,C,2), noattack(A,D,3),
    noattack(B,C,1), noattack(B,D,2), noattack(C,D,1).
GOAL: solution(A,B,C,D).
```
Solutions: `(2,4,1,3)` and `(3,1,4,2)`. `A,B,C,D` are the queen's column in rows 1–4. `noattack(X1,X2,D)`: `X1<>X2` blocks same-column, `abs(X1-X2)<>D` blocks the diagonal (D = the row-distance between that pair — 1 for adjacent rows, 2 for two rows apart, etc.).

**Lab Sheet 5's official version is actually 8-Queens** — same pattern, just 8 variables and `noattack` calls for all C(8,2)=28 pairs (tedious to write out fully in an exam; state the pattern and do a few pairs if asked to demonstrate, or use the Python backtracking version from `07_nqueens_python.md` if coding is allowed instead).

### Map Colouring (Australia)
```prolog
colour(WA,NT,SA,Q,NSW,V,T):-
    member(WA,[red,green,blue]), member(NT,[red,green,blue]),
    member(SA,[red,green,blue]), member(Q,[red,green,blue]),
    member(NSW,[red,green,blue]), member(V,[red,green,blue]),
    member(T,[red,green,blue]),
    WA<>NT, WA<>SA, NT<>SA, NT<>Q, SA<>Q, SA<>NSW,
    SA<>V, Q<>NSW, NSW<>V.
GOAL: colour(WA,NT,SA,Q,NSW,V,T).
```
Solution: `WA=red, NT=green, SA=blue, Q=red, NSW=green, V=red, T=`any (Tasmania is an island, no constraints).

---

## Style B — Modern `clpfd` (SWI-Prolog's Constraint Logic Programming over Finite Domains)
Cleaner and faster; this is what the actual Lab 5 report used and tested successfully. Requires `:- use_module(library(clpfd)).` at the top (built into SWI-Prolog, no separate install).

### SEND + MORE = MONEY, clpfd style
```prolog
:- use_module(library(clpfd)).

solution(S,E,N,D,M,O,R,Y):-
    Vars = [S,E,N,D,M,O,R,Y],
    Vars ins 0..9,
    all_different(Vars),
    S #\= 0, M #\= 0,
    M #= 1,
    D+E #= Y+10*C1,
    N+R+C1 #= E+10*C2,
    E+O+C2 #= N+10*C3,
    S+M+C3 #= O+10*C4,
    C4 #= 1,
    label(Vars).
```
Run: `swipl -q -t "solution(S,E,N,D,M,O,R,Y), writeln([S,E,N,D,M,O,R,Y])." file.pl` → `[9,5,6,7,1,0,8,2]` (matches Style A's answer).

Key differences from classical style: `ins 0..9` declares a whole domain at once (no `member` loop), `all_different/1` replaces the giant chain of `<>` pairs, `#=`/`#\=` are *constraint* operators (checked incrementally via propagation, not brute-force generate-then-test), and `label/1` triggers the actual search at the end. **This is faster and far less code to type live** — use it if you know the exam machine has SWI-Prolog.

### Map Colouring, clpfd style
```prolog
:- use_module(library(clpfd)).

solution(Colouring):-
    Colouring = [wa-WA, nt-NT, q-Q, nsw-NSW, v-V, sa-SA, t-T],
    Colours = [WA,NT,Q,NSW,V,SA,T],
    Colours ins 1..3,
    WA #\= NT, WA #\= SA, NT #\= Q, NT #\= SA,
    Q #\= NSW, Q #\= SA, NSW #\= V, NSW #\= SA, V #\= SA,
    label(Colours).
```
(map colours 1=red, 2=green, 3=blue). Verified output: `wa=red, nt=green, q=red, nsw=green, v=red, sa=blue, t=red`.

---

## Bonus: Map Colouring in plain Python (brute force, in case Prolog isn't available)
```python
from itertools import product

REGIONS = ["WA","NT","Q","NSW","V","SA","T"]
COLOURS = ["red","green","blue"]
ADJACENT = {
    "WA": ["NT","SA"], "NT": ["WA","Q","SA"], "Q": ["NT","NSW","SA"],
    "NSW": ["Q","V","SA"], "V": ["NSW","SA"], "SA": ["WA","NT","Q","NSW","V"], "T": [],
}

def is_valid(assignment):
    for region, colour in assignment.items():
        for nbr in ADJACENT[region]:
            if assignment.get(nbr) == colour:
                return False
    return True

def solve():
    for combo in product(COLOURS, repeat=len(REGIONS)):
        assignment = dict(zip(REGIONS, combo))
        if is_valid(assignment):
            return assignment

print(solve())
# {'WA': 'red', 'NT': 'green', 'Q': 'red', 'NSW': 'green', 'V': 'red', 'SA': 'blue', 'T': 'red'}
```
