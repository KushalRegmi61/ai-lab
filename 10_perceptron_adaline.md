# Perceptron / Adaline — Single Neuron Learning (Python)

Source: **corrected** using Lab Sheet 7's actual formulas + Lab Report 7's real, executed results and source code. Confirmed exam topic.

## IMPORTANT correction vs a generic "Perceptron"
A classic **Perceptron Learning Rule** updates weights using the **thresholded prediction**: `update = α(t − y_pred)`.
**Adaline (what this lab actually asks for)** updates weights using the **raw net input**, before thresholding: `update = α(t − y_in)`. This is the **delta rule / LMS / Widrow-Hoff rule**. The threshold is only applied *after* training, to decide the class.

```
y_in = b + Σ xi*wi                          (raw weighted sum, BEFORE thresholding)
b(new) = b(old) + α(t − y_in)
wi(new) = wi(old) + α(t − y_in)*xi
```
If your exam question says "Adaline" or "delta rule," use this. If it explicitly says "Perceptron Learning Rule," use the thresholded-error version instead — know both, but default to Adaline since that's what Lab Sheet 7 and the CR both reference.

## Tested code (matches the lab's actual `adaline.py`, condensed)
```python
import numpy as np

def train_adaline(X, t, alpha=0.1, mode="unipolar", tol=1e-3, max_epochs=500, seed=0):
    rng = np.random.default_rng(seed)
    w = rng.uniform(-0.5, 0.5, size=X.shape[1])
    b = rng.uniform(-0.5, 0.5)

    def activate(y_in):
        if mode == "unipolar":
            return 1 if y_in > 0.5 else 0
        return 1 if y_in >= 0 else -1

    for epoch in range(1, max_epochs + 1):
        max_dw = 0
        for xi, ti in zip(X, t):
            y_in = b + np.dot(w, xi)
            error = ti - y_in                 # <-- RAW error, not thresholded (this IS Adaline)
            dw = alpha * error * xi
            db = alpha * error
            w += dw
            b += db
            max_dw = max(max_dw, np.max(np.abs(dw)), abs(db))
        if max_dw < tol:
            break
    return w, b, epoch

# Unipolar AND
X = np.array([[0,0],[0,1],[1,0],[1,1]])
t = np.array([0,0,0,1])
w, b, epochs = train_adaline(X, t, alpha=0.1, mode="unipolar", seed=1)
print("weights:", w, "bias:", b, "epochs:", epochs)
```

## Real results from the lab (verified, actually run)
### Unipolar (α=0.1)
| Gate | w1 | w2 | b | Epochs to converge |
|---|---|---|---|---|
| AND | 0.556 | 0.528 | −0.278 | 2 |
| OR | 0.444 | 0.472 | 0.278 | 4 |
| NAND | −0.556 | −0.528 | 1.278 | 12 |
| NOR | −0.444 | −0.472 | 0.722 | 22 |

NAND/NOR take visibly more epochs than AND/OR — their decision boundary sits closer to a corner of the input square, so the bias needs more updates to move far enough from its small random start.

### Bipolar (−1/+1 encoding, α=0.1)
| Gate | w1 | w2 | b | Epochs |
|---|---|---|---|---|
| AND | 0.529 | 0.559 | −0.5 | 1 |
| OR | 0.471 | 0.441 | 0.5 | 2 |
| NAND | −0.529 | −0.559 | 0.5 | 2 |
| NOR | −0.471 | −0.441 | −0.5 | 3 |

**Bipolar converges much faster than unipolar (1–3 epochs vs 2–22).** Reason: with unipolar encoding, `0` is a "zero" input, so `Δwi = α(t−y_in)*xi = 0` whenever that feature is off — that weight only learns from patterns where the input is active. In bipolar, `-1` is a *nonzero* input, so every pattern updates every weight.

### Effect of learning rate (parameter study, real result)
U-shaped: `α=0.01` is slow (~57 epochs), `α≈0.1–0.3` is fastest (~4 epochs), `α=0.9` **oscillates and never converges** within 1000 epochs. **Moderate learning rate is best — too small is slow, too large overshoots.**

## McCulloch-Pitts fixed-weight gates (no learning — hand-designed thresholds)
| Gate | w1 | w2 | Threshold θ (fire if y_in ≥ θ) |
|---|---|---|---|
| AND | 1 | 1 | 2 |
| OR | 1 | 1 | 1 |
| NAND | −1 | −1 | −1 |
| NOR | −1 | −1 | 0 |

## Say out loud
- "Can a single Adaline/Perceptron learn XOR?" → **No** — XOR isn't linearly separable, no single straight line separates its 0s from its 1s. Needs a hidden layer (see `11_backpropagation_xor.md`) or a hand-built multi-gate McCulloch-Pitts network.
- "Why does bipolar converge faster?" → every input contributes to every weight update, unlike unipolar where a 0 input silences that weight's update for that pattern.
