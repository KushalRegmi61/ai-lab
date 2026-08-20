# Naive Bayes & Logistic Regression — ML Pipeline (Python, scikit-learn)

Source: **corrected/rebuilt** to match Lab Sheet 8's actual pipeline and Lab Report 8's real executed results exactly. Confirmed exam topic.

## The core ideas
- **Naive Bayes** (Gaussian NB) is **generative**: models `p(x|class)` via Bayes' rule, assumes features are conditionally independent given the class (the "naive" assumption).
- **Logistic Regression** is **discriminative**: models `p(class|x)` directly by fitting a decision boundary, trained by minimizing cross-entropy (log) loss.
- **Model capacity**: too little → **underfitting** (high bias, poor on both train and validation). Too much → **overfitting** (high variance, great on train, worse on validation).
- **Cross-entropy / log loss**: `Cost = -y*log(h(x)) - (1-y)*log(1-h(x))`. Confident-correct predictions cost ≈0; confident-wrong predictions cost →∞.
- **MLE** picks parameters maximizing the likelihood of observed data alone. **MAP** additionally multiplies in a prior over parameters — in Logistic Regression this is exactly what L2 regularization (`C` parameter) does.

## The full pipeline (matches the lab's exact setup — memorize this split pattern)
```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.naive_bayes import GaussianNB
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score, log_loss

iris = load_iris()
X, y = iris.data, iris.target

# STEP 1: carve out 20% as the final, untouched test set
X_temp, X_test, y_temp, y_test = train_test_split(
    X, y, test_size=0.20, random_state=42, stratify=y)
# STEP 2: split the remaining 80% into train (60% of total) and validation (20% of total)
X_train, X_val, y_train, y_val = train_test_split(
    X_temp, y_temp, test_size=0.25, random_state=42, stratify=y_temp)

scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)   # fit ONLY on train
X_val = scaler.transform(X_val)
X_test = scaler.transform(X_test)
```
**Why a 3-way split, not 2-way?** Train fits parameters. Validation picks the model/hyperparameters. Test is touched exactly once, at the very end, for an honest unbiased estimate — using test accuracy to pick hyperparameters would leak information and make the reported score falsely optimistic.

### Assignment 2 — Naive Bayes vs Logistic Regression
```python
gnb = GaussianNB().fit(X_train, y_train)
print("NB val acc:", accuracy_score(y_val, gnb.predict(X_val)))

logreg = LogisticRegression(max_iter=1000, random_state=42).fit(X_train, y_train)
print("LogReg val acc:", accuracy_score(y_val, logreg.predict(X_val)))
```
**Real result: both tie at 0.933 (28/30).** Expected for Iris — its 3 classes are close to linearly separable and roughly Gaussian per-class, exactly the regime where a simple generative model and a linear discriminative model behave similarly.

### Assignment 3 — Model capacity (Decision Tree, varying `max_depth`)
```python
for d in [1, 3, 5, None]:
    tree = DecisionTreeClassifier(max_depth=d, random_state=42).fit(X_train, y_train)
    tr_acc = accuracy_score(y_train, tree.predict(X_train))
    va_acc = accuracy_score(y_val, tree.predict(X_val))
    print(d, tr_acc, va_acc)
```
**Real results:**
| max_depth | Train acc | Val acc |
|---|---|---|
| 1 | 0.667 | 0.667 |
| 3 | 0.989 | 0.900 |
| 5 | 1.000 | 0.933 |
| None | 1.000 | 0.933 |

- **`max_depth=1` underfits**: train AND val accuracy both low and nearly equal — the model is too simple even for the training data (high bias).
- **`max_depth=None` overfits**: 100% train accuracy (memorized every example) but val accuracy is measurably lower (high variance) — and note `max_depth=5` reaches the *same* val accuracy with a far smaller tree, proving the extra capacity of the unrestricted tree buys nothing.

### Assignment 4 — Selecting a hyperparameter, then ONE final test evaluation
```python
best_depth = 5   # tied with None on validation; pick the simpler model (Occam's razor)
final_tree = DecisionTreeClassifier(max_depth=best_depth, random_state=42).fit(X_train, y_train)
test_acc = accuracy_score(y_test, final_tree.predict(X_test))
print(test_acc)   # 0.933 -- evaluated ONLY ONCE, on the untouched test set
```

### Assignment 6 — Cross-entropy / log loss
```python
probs = logreg.predict_proba(X_val)
val_logloss = log_loss(y_val, probs)   # 0.215 in the real run
```
A confidently-correct sample (p(true class)=0.997) contributes loss ≈0.003. A confidently-wrong sample (p(true class)=0.262, model favored the wrong class) contributes loss ≈1.340 — **~390× larger**. This asymmetry is the whole point of log loss: barely penalize confident-correct, heavily penalize confident-wrong.

### Assignment 7 — MLE vs MAP (regularization strength `C`)
```python
for C in [0.001, 0.01, 0.1, 1, 10, 100, 1000]:
    m = LogisticRegression(C=C, max_iter=2000, random_state=42).fit(X_train, y_train)
    print(C, np.linalg.norm(m.coef_), accuracy_score(y_val, m.predict(X_val)))
```
Small `C` = strong prior = MAP-like (shrinks weights, can underfit — val acc drops to 0.80 at C≤0.01). Large `C` = weak/no prior = MLE-like (weights grow freely, fits training distribution more closely). `1/C` is itself a hyperparameter tuned via validation, same as `max_depth`.

## The 4-line pattern to memorize for ANY sklearn classifier
```python
model = SomeClassifier(...)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
accuracy_score(y_test, predictions)
```
Swap line 1's class name — `GaussianNB`, `LogisticRegression`, `DecisionTreeClassifier` — everything else stays identical.

## Say out loud (the lab's actual "Questions to be Answered")
- **High train, lower val accuracy?** → overfitting: the model memorized training-specific noise that doesn't generalize.
- **Why is validation useful for hyperparameters?** → estimates generalization without touching the test set; tuning against test directly would bias the final reported accuracy.
- **Underfitting ↔ high bias ↔ low capacity**: a too-simple model makes systematic errors it can't fix no matter how much data it sees.
- **Overfitting ↔ high variance ↔ high capacity**: a too-flexible model fits noise specific to the training sample; a different sample would yield a different model.
- **MAP vs MLE?** → MAP adds a prior `p(θ)` over parameters (e.g. "smaller weights are more plausible"); MLE only uses the likelihood of the data, equivalent to MAP with a flat/uninformative prior.
- **Why does cross-entropy punish confident-wrong so hard?** → `-log(p)` diverges to infinity as `p→0`.
