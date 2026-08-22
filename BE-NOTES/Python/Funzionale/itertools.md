---
topic: "itertools — Python"
tags: [python, functional, itertools, iteration]
nav_prev: "[[Lambda map filter reduce.md]]"
nav_next: "[[functools.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/itertools.html](https://docs.python.org/3/library/itertools.html)

Modulo built-in per iteratori lazy e combinatoria. Le funzioni di itertools sono **lazy** (non creano liste intermedie), il che le rende efficienti in memoria per grandi dataset.

Ideale in combinazione con [[BE-NOTES/Python/Funzionale/Lambda map filter reduce|lambda]] e [[BE-NOTES/Python/Funzionale/functools|functools]] per pipeline funzionali senza allocazioni intermedie.

Performance: `itertools` è scritto in C — spesso più veloce di loop manuali e list comprehension per operazioni su grandi sequenze.

```python
from itertools import count, cycle, repeat

# count(start, step) — contatore infinito
for i in count(0, 2):      # 0, 2, 4, 6, ...
    if i > 10:
        break

# cycle(iterable) — ripete all'infinito
for colore in cycle(["rosso", "verde"]):
    if condizione: break

# repeat(element, times) — ripete n volte
list(repeat("x", 3))  # ["x", "x", "x"]
```

## Combinatoria

```python
from itertools import product, permutations, combinations

# product — prodotto cartesiano
list(product([1, 2], ["a", "b"]))
# [(1,"a"), (1,"b"), (2,"a"), (2,"b")]

# permutations — permutazioni (ordine conta)
list(permutations([1, 2, 3], 2))
# [(1,2), (1,3), (2,1), (2,3), (3,1), (3,2)]

# combinations — combinazioni (ordine non conta)
list(combinations([1, 2, 3], 2))
# [(1,2), (1,3), (2,3)]

# combinations_with_replacement
list(combinations_with_replacement([1, 2], 2))
# [(1,1), (1,2), (2,2)]
```

## Raggruppamento e slicing

```python
from itertools import groupby, islice, chain

# groupby — raggruppa elementi consecutivi
dati = [("a", 1), ("a", 2), ("b", 3), ("b", 4)]
for chiave, gruppo in groupby(dati, key=lambda x: x[0]):
    print(chiave, list(gruppo))  # "a" [(a,1),(a,2)], "b" [(b,3),(b,4)]

# islice — come slice ma su iteratori
list(islice(range(10), 2, 8, 2))  # [2, 4, 6]

# chain — concatena iterabili
list(chain([1, 2], [3, 4], [5]))  # [1, 2, 3, 4, 5]

# pairwise — coppie consecutive (Python 3.10+)
from itertools import pairwise
list(pairwise([1, 2, 3, 4]))  # [(1,2), (2,3), (3,4)]
```

## Accumulate e compress

```python
from itertools import accumulate, compress
import operator

# accumulate — somma cumulativa (o altra operazione)
list(accumulate([1, 2, 3, 4]))           # [1, 3, 6, 10]
list(accumulate([1, 2, 3, 4], operator.mul))  # [1, 2, 6, 24]

# compress — filtra con selettore booleano
list(compress("ABCDEF", [1, 0, 1, 0, 1, 0]))  # ["A", "C", "E"]
```
