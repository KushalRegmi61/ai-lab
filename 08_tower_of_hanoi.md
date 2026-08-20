# Tower of Hanoi (Python)

Source: this session's teaching pass, code verified by execution. Confirmed exam topic. **Not in any lab sheet — built from scratch.**

## The problem
Move N disks from `source` to `target` using `aux` as helper, one disk at a time, never a bigger disk on a smaller one.

## The recursive insight (this IS the algorithm)
To move N disks from source to target:
1. Move top N-1 disks from source to **aux** (using target as *that* sub-problem's helper).
2. Move the single largest disk from source to target.
3. Move the N-1 disks from aux to target (using source as *that* sub-problem's helper).

## Tested code
```python
def hanoi(n, source, aux, target, moves):
    if n == 1:
        moves.append(f"Move disk 1 from {source} to {target}")
        return
    hanoi(n-1, source, target, aux, moves)   # step 1
    moves.append(f"Move disk {n} from {source} to {target}")  # step 2
    hanoi(n-1, aux, source, target, moves)   # step 3

moves = []
hanoi(3, 'A', 'B', 'C', moves)
for m in moves: print(m)
print(f"Total moves: {len(moves)}")   # 7 = 2^3 - 1
```
Output for n=3:
```
Move disk 1 from A to C
Move disk 2 from A to B
Move disk 1 from C to B
Move disk 3 from A to C
Move disk 1 from B to A
Move disk 2 from B to C
Move disk 1 from A to C
Total moves: 7
```

## Why the peg arguments swap between the two recursive calls
```python
hanoi(n-1, source, target, aux, moves)   # target and aux SWAPPED
hanoi(n-1, aux, source, target, moves)   # source and aux SWAPPED
```
Each recursive call relabels which peg plays "source/aux/target" for *that* smaller sub-problem — e.g. in step 1, the real target is temporarily just scratch space, so it becomes the sub-problem's helper.

## Say out loud
- "Minimum moves for N disks?" → `2^N - 1`.
- "Why recursive?" → the problem is self-similar: "move N disks" reduces to "move N-1 disks" (twice) + one single move — the textbook shape of recursion.
