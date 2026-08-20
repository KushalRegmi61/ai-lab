# N-Queens — Backtracking (Python)

Source: this session's teaching pass + Lab Sheet 5 (specifies 8-Queens formally). Same problem as the Prolog CSP version, different technique.

## Representation trick
`board[row] = column` — one queen per row by construction, so row conflicts are impossible; only check column/diagonal against earlier rows.

## Tested code
```python
def solve_nqueens(n):
    solutions = []
    board = [-1] * n

    def is_safe(row, col):
        for r in range(row):
            c = board[r]
            if c == col or abs(c - col) == abs(r - row):
                return False
        return True

    def backtrack(row):
        if row == n:
            solutions.append(board.copy())
            return
        for col in range(n):
            if is_safe(row, col):
                board[row] = col
                backtrack(row + 1)
                board[row] = -1          # UNDO before trying next column
    backtrack(0)
    return solutions

print(solve_nqueens(4))   # [[1,3,0,2],[2,0,3,1]]
print(len(solve_nqueens(8)))  # 92 solutions -- Lab Sheet 5's official N=8 version
```

## The diagonal check
`abs(c - col) == abs(r - row)`: two positions share a diagonal exactly when column-distance equals row-distance.

## Mapping to the Prolog version (side by side)
| Prolog | Python |
|---|---|
| `member(A,[1..n])` | `for col in range(n)` |
| `noattack(A,B,D)` constraint check | `is_safe(row, col)` |
| automatic backtracking on failure | **explicit** `board[row] = -1` after the recursive call — the one thing Prolog hides that Python makes you write |

## Say out loud
"Forgetting the undo line (`board[row] = -1`) is the single most common bug — without it, stale queens from a failed branch silently corrupt later checks."
