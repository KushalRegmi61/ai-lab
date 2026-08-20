# Prolog Basics — Facts, Rules, Lists, Recursion

Source: Lab Sheet 1, Lab Report 1.

## The core idea
Prolog is **declarative**: state facts and rules describing what's true, then ask a goal — Prolog searches for a proof using unification and backtracking.

## Syntax
| Concept | Syntax | Example |
|---|---|---|
| Atom (fixed value) | lowercase start, or quoted | `john`, `'Ram'` |
| Variable | Uppercase or `_` start | `X`, `Result`, `_weight` |
| Fact | `name(args).` | `parent(john, mary).` |
| Rule | `head :- body.` | `father(F,C):- parent(F,C), male(F).` |
| Anonymous variable | `_` | used when a value isn't needed |
| Conjunction (AND) | `,` | `parent(P,A), parent(P,B)` |
| Disjunction (OR) | `;` | `H = 1 ; H = 2` |
| Not unifiable | `\=` | `X \= Y` |
| Numerically not equal | `=\=` | `A =\= B` (**not** `<>`, see below) |
| Terms not identical | `\==` | `red \== green` |
| Cut (stop backtracking) | `!` | used for efficiency |

> ### ⚠️ `<>` is not SWI-Prolog
> `<>` for "not equal" is **Turbo/Visual Prolog** only. On SWI-Prolog it is a syntax error. Use **`=\=` for numbers**, **`\==` for atoms**, **`\=` for general non-unifiability**. See the operator table in `03_prolog_csp.md`.

A variable's scope is a **single clause** — the same variable name in two different rules means two different things.

## Lists
```prolog
[ram, shyam, hari, sita]        % a list
[H|T]                            % Head/Tail split: H=ram, T=[shyam,hari,sita]
```
Lists are internally binary trees: head = first element, tail = rest of the list. Every recursive list predicate follows this shape:
```prolog
predicate([], BaseCaseResult).
predicate([H|T], Result):- predicate(T, TailResult), <combine H with TailResult>.
```

## Worked example — HCF (Euclidean algorithm)
```prolog
hcf(X, 0, X):- X > 0.
hcf(X, Y, R):-
    Y > 0,
    Rem is X mod Y,
    hcf(Y, Rem, R).
```
`hcf(24, 36, H).` → `H = 12`. Base case: remainder 0 means the last non-zero remainder is the HCF. `is` evaluates the arithmetic expression on its right and binds it to the variable on the left — you cannot write `Rem = X mod Y` (that just fails to unify a compound term), you must use `is`.

## Worked example — Family relationships
```prolog
parent(john, mary).  parent(john, tom).
parent(mary, anna).  parent(mary, peter).
male(john). male(tom). female(mary). female(anna).

father(F,C):- parent(F,C), male(F).
mother(M,C):- parent(M,C), female(M).
grandparent(GP,GC):- parent(GP,P), parent(P,GC).
sibling(A,B):- parent(P,A), parent(P,B), A \= B.
ancestor(A,D):- parent(A,D).
ancestor(A,D):- parent(A,X), ancestor(X,D).      % recursive case
```
`ancestor/2` is the classic "recursive closure" pattern: base case = direct parent, recursive case = parent of an ancestor. This pattern generalizes to any "reachability" question (e.g., "can node A reach node B" in a graph).

## List-processing assignments (Lab Sheet 1's actual assignment list)
**Sum of a list:**
```prolog
sum_list([], 0).
sum_list([H|T], Sum):- sum_list(T, TailSum), Sum is H + TailSum.
```
**Length of a list:**
```prolog
list_length([], 0).
list_length([_|T], Len):- list_length(T, TailLen), Len is TailLen + 1.
```
**Append two lists:**
```prolog
append_lists([], L, L).
append_lists([H|T], L2, [H|Result]):- append_lists(T, L2, Result).
```
**Filter a list (keep only 1s and 2s):**
```prolog
filter_ones_twos([], []).
filter_ones_twos([H|T], [H|R]):- (H = 1 ; H = 2), filter_ones_twos(T, R).
filter_ones_twos([H|T], R):- H \= 1, H \= 2, filter_ones_twos(T, R).
```
**Delete all occurrences of an element:**
```prolog
delete_element(_, [], []).
delete_element(X, [X|T], R):- !, delete_element(X, T, R).      % cut: commit to this match
delete_element(X, [H|T], [H|R]):- delete_element(X, T, R).
```
The `!` (cut) in `delete_element` stops Prolog from also trying the third clause once the second clause's head matches — without it, Prolog might produce duplicate/wrong solutions on backtracking.

## Say out loud, if asked "what's different about Prolog?"
> "Prolog is declarative — we describe knowledge as facts and rules, then ask goals, and Prolog searches for a proof via unification and backtracking. Procedural languages like C/Python instead require you to write explicit step-by-step control flow."

## Running these on this laptop
SWI-Prolog **9.0.4** is installed at `/usr/bin/swipl`, and the VS Code extension **New VSC-Prolog** (`amauryrabouan.new-vsc-prolog`) is already configured for it.

```bash
swipl family.pl          # load file, land in the ?- REPL
```
At the `?-` prompt: press **`;`** (a keypress, not typed+Enter) for the next solution, **Enter** to stop. Useful commands: `make.` (reload after editing — use constantly), `listing(father).`, `trace.` / `nodebug.`, `halt.`

One-shot, no REPL:
```bash
swipl -q -g "father(john,X), writeln(X)" -t halt family.pl
```
VS Code shortcuts: `Alt+X L` load document, `Alt+X Q` query goal under cursor, `F8` next error.

**Most common errors:** missing final period (the error points at the *next* clause), and forgetting `make.` after an edit.
