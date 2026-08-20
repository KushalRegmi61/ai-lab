# Minimax and Alpha-Beta Pruning (Python)

Source: this session's teaching pass, code verified by execution. Confirmed exam topic. **Not in any lab sheet — built from scratch, so this is genuinely new material to you.**

## The core idea
For two-player, zero-sum games (Tic-Tac-Toe, Chess). Build a game tree; leaves have an evaluation score. **MAX** picks the highest value among children, **MIN** picks the lowest, alternating by depth. **Alpha-Beta** gives the identical answer while skipping provably-irrelevant branches.

## Worked example — classic depth-3 tree
```
                     MAX (root)
              /                    \
           MIN                     MIN
         /      \                /      \
       MAX       MAX           MAX       MAX
      /   \     /   \         /   \     /   \
     3     5   2     9       12    5   23    23
```
```
Leftmost MAX: max(3,5)=5        Next MAX: max(2,9)=9      Left MIN: min(5,9)=5
Third MAX: max(12,5)=12         Fourth MAX: max(23,23)=23  Right MIN: min(12,23)=12
Root MAX: max(5,12) = 12
```

## Tested code
```python
import math

def minimax(depth, node_index, is_max, scores, max_depth):
    if depth == max_depth:
        return scores[node_index]
    if is_max:
        best = -math.inf
        for i in range(2):
            val = minimax(depth+1, node_index*2+i, False, scores, max_depth)
            best = max(best, val)
        return best
    else:
        best = math.inf
        for i in range(2):
            val = minimax(depth+1, node_index*2+i, True, scores, max_depth)
            best = min(best, val)
        return best

def alphabeta(depth, node_index, is_max, scores, max_depth, alpha, beta):
    if depth == max_depth:
        return scores[node_index]
    if is_max:
        best = -math.inf
        for i in range(2):
            val = alphabeta(depth+1, node_index*2+i, False, scores, max_depth, alpha, beta)
            best = max(best, val)
            alpha = max(alpha, best)
            if beta <= alpha: break          # PRUNE
        return best
    else:
        best = math.inf
        for i in range(2):
            val = alphabeta(depth+1, node_index*2+i, True, scores, max_depth, alpha, beta)
            best = min(best, val)
            beta = min(beta, best)
            if beta <= alpha: break          # PRUNE
        return best

scores = [3, 5, 2, 9, 12, 5, 23, 23]
max_depth = int(math.log2(len(scores)))
print("Minimax:", minimax(0, 0, True, scores, max_depth))                          # 12
print("Alpha-Beta:", alphabeta(0, 0, True, scores, max_depth, -math.inf, math.inf)) # 12
```

## Alpha-Beta explained
- `alpha` = the best value MAX can already guarantee elsewhere in the tree.
- `beta` = the best value MIN can already guarantee elsewhere.
- If `beta <= alpha` at any point: **stop exploring this branch**. Meaning: MIN already has a way to hold this branch down to `beta`, but MAX already has a guaranteed `alpha` at least as good elsewhere — neither rational player would let the game actually reach here.

`node_index*2+i` is the "array-as-binary-tree" indexing trick: doubling at each level plus 0/1 for left/right child computes the exact leaf index by the time `depth==max_depth`.

## If asked for a concrete game
Say **Tic-Tac-Toe**: leaves = board evaluation (+1 you win, -1 you lose, 0 draw), MAX = your turn, MIN = opponent's turn, tree depth = remaining empty squares.

## Say out loud
- "Why does Alpha-Beta give the identical answer?" → it never prunes a branch that could change the final decision, only branches mathematically proven irrelevant.
- "Best-case speedup?" → with perfect move-ordering, Alpha-Beta can search twice as deep in the same time as plain Minimax.
