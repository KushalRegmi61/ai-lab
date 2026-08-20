# Backpropagation Neural Network — XOR (Python)

Source: **corrected/verified** against Lab Sheet 7's formulas and Lab Report 7's real executed results. Confirmed exam topic.

## Why XOR needs more than one neuron
XOR isn't linearly separable — no single straight line separates its 0s from its 1s. Two fixes exist: (A) hand-build a fixed-weight network from simpler gates, or (B) train a hidden layer with backpropagation.

## Fix A — Hand-built McCulloch-Pitts network (fixed weights, no training)
```
XOR(x1, x2) = AND( OR(x1,x2), NAND(x1,x2) )
```
Each unit: `y = 1 if y_in >= θ else 0`, `y_in = Σ wi*xi`.
```
z1 = OR(x1,x2):   w=(1,1),   θ1=1
z2 = NAND(x1,x2): w=(-1,-1), θ2=-1
y  = AND(z1,z2):  w=(1,1),   θ=2
```
```python
def mp_unit(x, w, theta):
    y_in = sum(xi*wi for xi, wi in zip(x, w))
    return 1 if y_in >= theta else 0

def xor_mcp(x1, x2):
    z1 = mp_unit([x1,x2], [1,1], 1)      # OR
    z2 = mp_unit([x1,x2], [-1,-1], -1)   # NAND
    y  = mp_unit([z1,z2], [1,1], 2)      # AND
    return y

for a in (0,1):
    for b in (0,1):
        print(a, b, "->", xor_mcp(a,b))
# 0 0 -> 0   0 1 -> 1   1 0 -> 1   1 1 -> 0   (exact XOR table)
```

## Fix B — 2-2-1 MLP trained with Backpropagation (the general/learned solution)
Sigmoid activation `f(x)=1/(1+e^-x)`, `f'(x)=f(x)(1-f(x))`. Per-pattern update:
```
δk = (tk − yk) f'(y_in_k)                    # output layer error
δ_in_j = Σ δk * w_jk                          # propagate back to hidden layer
δj = δ_in_j * f'(z_in_j)
Δw_jk = α δk zj      Δw_ij = α δj xi
w(new) = w(old) + Δw
```

### Tested code
```python
import numpy as np
np.random.seed(42)

X = np.array([[0,0],[0,1],[1,0],[1,1]])
y = np.array([[0],[1],[1],[0]])

def sigmoid(x): return 1/(1+np.exp(-x))
def sigmoid_deriv(x): return x*(1-x)

W1 = np.random.uniform(-1,1,(2,4)); b1 = np.zeros((1,4))
W2 = np.random.uniform(-1,1,(4,1)); b2 = np.zeros((1,1))
lr = 0.5

for epoch in range(10000):
    # forward pass
    hidden_out = sigmoid(np.dot(X, W1) + b1)
    final_out  = sigmoid(np.dot(hidden_out, W2) + b2)
    # backward pass
    error = y - final_out
    d_output = error * sigmoid_deriv(final_out)
    error_hidden = d_output.dot(W2.T)
    d_hidden = error_hidden * sigmoid_deriv(hidden_out)
    # weight updates (gradient descent)
    W2 += hidden_out.T.dot(d_output) * lr
    b2 += np.sum(d_output, axis=0, keepdims=True) * lr
    W1 += X.T.dot(d_hidden) * lr
    b1 += np.sum(d_hidden, axis=0, keepdims=True) * lr

for xi, target in zip(X, y):
    out = sigmoid(np.dot(sigmoid(np.dot(xi, W1)+b1), W2)+b2)
    print(xi, "-> predicted:", round(float(out[0][0]),3), "actual:", target[0])
# [0,0]->0.01  [0,1]->0.98  [1,0]->0.99  [1,1]->0.02   (all correct)
```

## Real results from the lab (verified, actually run — 20000 epochs, α=0.5)
Final weights (one converged run): `v11=6.48, v21=-6.50, vb1=-3.53, v12=6.25, v22=-6.00, vb2=2.99, w1=10.73, w2=-10.00, wb=4.78`. Final MSE ≈ 1.3×10⁻⁴.
| x1 | x2 | raw output | predicted |
|---|---|---|---|
| 0 | 0 | 0.012 | 0 |
| 0 | 1 | 0.987 | 1 |
| 1 | 0 | 0.989 | 1 |
| 1 | 1 | 0.010 | 0 |

**Random initialization matters:** of 20 random seeds tried in the actual lab, 13 converged correctly, 7 got stuck in a local minimum (MSE≈0.13) — a known difficulty of training tiny MLPs on XOR by gradient descent. If your code doesn't converge, **try a different random seed**, don't assume the code is wrong.

## Say out loud
- "Why can't one perceptron do XOR?" → not linearly separable.
- "What does 'backpropagation' mean, literally?" → forward pass computes predictions; the error signal is then propagated *backward*, layer by layer, and each weight is updated proportionally to how much it contributed to the error (chain rule from calculus, applied layer by layer).
- "What's the practical fix for a hidden layer alone not being enough?" → nothing extra needed, XOR just needs ≥1 hidden layer + backprop; deeper problems may need more hidden units or layers.
