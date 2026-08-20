# Agent Types — Simple Reflex vs Model-Based (Vacuum World)

Source: Lab Sheet 3, Lab Report 3. **Not named by the CR as confirmed exam scope** — lower priority, read only if time remains.

## The core idea
- **Simple Reflex Agent**: acts only on the *current* percept via condition-action rules. No memory.
- **Model-Based Agent**: maintains an internal `world_model` (memory), updates it each percept, and can reason about parts of the environment it isn't currently observing.

## Vacuum World setup
Two rooms (A, B), each either "Clean" or "Dirty". Agent is in one room at a time.

## Simple Reflex Agent (tested code, from the lab report)
```python
def simple_reflex_agent(location, status):
    if status == "Dirty":
        return "Suck"
    elif location == "A":
        return "Move to B"
    elif location == "B":
        return "Move to A"
    else:
        return "NoOp"
```
No memory — after cleaning A, if it later perceives `(A, Clean)` it still moves to B, unaware A is already clean.

## Model-Based Agent (tested code, from the lab report)
```python
world_model = {"A": "Clean", "B": "Clean"}

def model_based_agent(location, status):
    world_model[location] = status
    if world_model["A"] == "Dirty":
        world_model["A"] = "Clean"
        return "Go to A and Suck"
    elif world_model["B"] == "Dirty":
        world_model["B"] = "Clean"
        return "Go to B and Suck"
    else:
        return "Do Nothing, all are clean"
```
Because it remembers both rooms' status, it avoids redundant moves — this is the key advantage over Simple Reflex.

## Comparison table
| | Simple Reflex | Model-Based |
|---|---|---|
| Memory | None | `world_model` dict, persists across calls |
| Decision basis | current percept only | current percept + stored state |
| Room A already clean | Moves to B anyway | Does nothing (remembers) |
| Efficiency | May revisit clean rooms | Avoids unnecessary actions |

## Say out loud
"A Simple Reflex Agent maps percept → action directly via condition-action rules; a Model-Based Agent additionally maintains and updates an internal model of the world, letting it reason about parts of the environment not currently observed."
