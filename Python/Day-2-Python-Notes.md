# Table-of-Contents

<!-- toc -->

- [Random-Module](#random-module)
      - [Important-Notes](#important-notes)
      - [Code-Snippet](#code-snippet)
      - [Key-Takeaway](#key-takeaway)

<!-- tocstop -->

---

# Random-Module
#### Important-Notes

* Provides random numbers, selections, and distributions.
* `random()` → float \[0.0, 1.0)
* `uniform(a, b)` → float in range \[a, b]
* `randint(a, b)` → int in range \[a, b]
* `randrange(start, stop, step)` → int with step
* `choice()` / `choices()` / `sample()` → pick items
* `shuffle()` → in-place shuffle of list
* `seed()` → reproducibility (fixed results)
* Has built-in probability distributions (`gauss`, `expovariate`, etc.)

#### Code-Snippet

```python
import random

# 1. Random Numbers
print(random.random())        # Output: 0.6394... (float between 0 and 1)
print(random.uniform(1, 5))   # Output: 3.74...  (float between 1 and 5)
print(random.randint(1, 10))  # Output: 7        (int between 1 and 10)
print(random.randrange(1, 10, 2))  # Output: 3  (odd number < 10)

# 2. Choice & Sampling
items = [1, 2, 3, 4, 5]
print(random.choice(items))        # Output: 4  (random element)
print(random.choices(items, k=3))  # Output: [2, 5, 5] (with replacement)
print(random.sample(items, 3))     # Output: [1, 4, 5] (unique elements)

# 3. Shuffle
random.shuffle(items)
print(items)  # Output: [3, 5, 1, 2, 4]

# 4. Seed
random.seed(42)
print(random.randint(1, 10))  # Output: 2 (fixed result)

# 5. Distributions
print(random.gauss(0, 1))        # Output: -0.1440... (Gaussian)
print(random.expovariate(1))     # Output: 0.234...   (Exponential)
print(random.betavariate(2, 5))  # Output: 0.145...   (Beta)
```

#### Key-Takeaway

* `random.random()` → float \[0.0, 1.0).
* `randint` / `uniform` → numbers in range.
* `choice` / `sample` → pick items.
* `shuffle` → reorder list.
* `seed` → reproducibility for debugging.
