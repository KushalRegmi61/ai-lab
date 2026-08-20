# Prolog — First Order Predicate Logic (FOPL) & Proving

Source: Lab Sheet 2, Lab Report 2.

## The core idea
FOPL extends propositional logic with **predicates**, **variables**, and **quantifiers**: ∀ ("for all"), ∃ ("there exists"). Prolog is a direct computational realization of a subset of FOPL (Horn clauses):

| FOPL | Prolog |
|---|---|
| `∀x (Man(x) ⇒ Mortal(x))` | `mortal(X):- man(X).` |
| Implication `⇒` | `:-` |
| Conjunction `∧` | `,` |
| Universally quantified variable | Prolog variable |
| Ground fact / `∃` assertion | Prolog fact |

**Backward chaining** (Prolog's default): start from the goal, work backward to facts via unification/backtracking.
**Forward chaining**: start from facts, apply rules forward until the goal is derived. Not built into Prolog — simulated with `assert/1`.

## Worked example — The "Criminal" problem (Lab Sheet 2's main worked example)
Statements: Every American who sells weapons to hostile nations is a criminal. Every enemy of America is hostile. Iraq is a country and an enemy of America. George is American and sells missiles to Iraq.
```prolog
criminal(X):- american(X), sells_missiles(X, Y), hostile(Y).
hostile(X):- country(X).
american("George").
sells_missiles("George", "Iraq").
country("Iraq").

GOAL: criminal("George").   % true
```
Resolution trace: George is American ✓, sells missiles to Iraq ✓, Iraq is a country → hostile ✓ → all three conditions of `criminal/1` hold → **George is a criminal.**

## Worked example — Steve and easy courses
Facts: Steve only likes easy courses; science courses are hard; basket-weaving courses are easy; BK301 is a basket-weaving course.
```prolog
course(bk301).
basket_weaving_course(bk301).

likes(steve, X):- course(X), easy(X).
hard(X):- science_course(X).
easy(X):- basket_weaving_course(X).

GOAL: likes(steve, X).      % X = bk301
```
**Resolution proof (write this out if asked to "prove by resolution"):**
1. `basket_weaving_course(bk301)` — given fact.
2. By rule `easy(X):-basket_weaving_course(X)`: `easy(bk301)`.
3. `course(bk301)` — given fact.
4. By rule `likes(steve,X):-course(X),easy(X)`: both hold for `bk301`.
5. ∴ `likes(steve, bk301)`. ∎

## Worked example — Horses, Mammals, and Charlie (a subtlety worth knowing)
Given facts: (1) Horses are mammals. (2) An offspring of a horse is a horse. (3) Bluebeard is Charlie's parent. (4) Offspring and parent are inverse relations. (5) Every mammal has a parent.
**Question: is Charlie a horse?**

**The subtlety:** the literal facts above never state that Bluebeard *is* a horse — so strictly, this is **not provable** from the given facts alone (a classic demonstration that resolution can't invent facts you didn't give it). If your exam slip gives you exactly these 5 statements and asks to prove `horse(charlie)`, the correct answer is: *it cannot be proven true; the knowledge base is insufficient.*

However, if the problem additionally supplies (or you're told to assume) `horse(bluebeard).` — as most textbook versions of this classic puzzle intend — then:
```prolog
parent(bluebeard, charlie).
horse(bluebeard).
mammal(X):- horse(X).
horse(X):- offspring(X, Y), horse(Y).
offspring(X, Y):- parent(Y, X).

GOAL: horse(charlie).   % true, IF horse(bluebeard) is given
```
Trace: `offspring(charlie, Y)` → `parent(Y, charlie)` → `Y = bluebeard` (inverse relation) → `horse(bluebeard)` is a fact → `horse(charlie)` succeeds by rule (2) → and then `mammal(charlie)` also succeeds by rule (1).

**Say out loud if asked:** "This demonstrates that Prolog's resolution is sound but only as complete as the facts you give it — without `horse(bluebeard)` explicitly stated or derivable, the goal correctly fails rather than guessing."

## Worked example — Student Evaluation (Forward vs Backward Chaining)
Rules: R1: `good_position(X) ∧ no_higher_marks(X) ⇒ first_position(X)`. R2: `attends_lectures(X) ∧ studies_books(X) ⇒ covers_course(X)`. R3: `covers_course(X) ⇒ good_position(X)`.
Facts: `attends_lectures(student)`, `studies_books(student)`, `no_higher_marks(student)`.

```prolog
attends_lectures(student). studies_books(student). no_higher_marks(student).
covers_course(X):- attends_lectures(X), studies_books(X).
good_position(X):- covers_course(X).
first_position(X):- good_position(X), no_higher_marks(X).

GOAL: first_position(student).   % true
```
**Backward chaining trace** (Prolog's default — goal-driven, top-down):
```
first_position(student)?
  need good_position(student) AND no_higher_marks(student)
    no_higher_marks(student) -> FACT, true
    good_position(student)?
      need covers_course(student)
        need attends_lectures(student) -> FACT, true
        need studies_books(student)    -> FACT, true
      => covers_course(student) TRUE
    => good_position(student) TRUE
=> first_position(student) TRUE
```
**Forward chaining trace** (facts-driven, bottom-up — simulate with `assert/1` since Prolog doesn't do this natively):
```
Step 1 (Rule II): attends_lectures + studies_books -> derive covers_course(student)
Step 2 (Rule III): covers_course -> derive good_position(student)
Step 3 (Rule I): good_position + no_higher_marks -> derive first_position(student)
```
Both strategies reach the same conclusion — that's expected for sound and complete Horn-clause reasoning; they just traverse the same rule graph in opposite directions.

## Worked example — Monkey-Banana Problem
Room has monkey, chair, bananas (out of reach). Monkey is dexterous, chair is tall, monkey can climb the chair, monkey can move the chair under the bananas.
```prolog
in_room(bananas). in_room(chair). in_room(monkey).
dexterous(monkey). tall(chair).
can_move(monkey, chair, bananas). can_climb(monkey, chair).

can_reach(X,Y):- dexterous(X), close(X,Y).
close(X,Z):- get_on(X,Y), under(Y,Z), tall(Y).
get_on(X,Y):- can_climb(X,Y).
under(Y,Z):- in_room(X), in_room(Y), in_room(Z), can_move(X,Y,Z).

GOAL: can_reach(monkey, bananas).   % true
```

## The general recipe for ANY FOPL word-problem question
1. Underline every noun → these become your predicates/atoms.
2. Find every "if... then" sentence → write as `head:- body.` (comma-join multiple conditions).
3. Write the plain assertions as facts.
4. Pose the goal with a free variable if the question asks "what/who...", or a ground query if it asks "is it true that...".
