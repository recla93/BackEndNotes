---
topic: "Lambda, map, filter, reduce — Python"
tags: [python, functional, lambda, higher-order]
nav_prev: "[[Concetti Base.md]]"
nav_next: "[[itertools.md]]"
---
Riferimento ufficiale: [docs.python.org/3/howto/functional.html](https://docs.python.org/3/howto/functional.html)

Prima di Python 3.0 `map`/`filter`/`reduce` erano onnipresenti. Oggi le **list comprehension** sono spesso più leggibili. Tuttavia, lambda e higher-order functions sono utili per:
- Callback (sorting, threading, asyncio)
- Pipe di trasformazione lazy
- Operazioni con librerie funzionali (toolz, returns)
- Casi dove una funzione completa `def` è overkill

Vedi anche: [[BE-NOTES/Python/Funzionale/Concetti Base|Concetti Base FP]], [[BE-NOTES/Python/Funzionale/functools|functools.partial]], [[BE-NOTES/Python/OOP/Decoratori|Decoratori]].

Espressione anonima inline, limitata a una singola espressione:

```python
quadrato = lambda x: x ** 2
quadrato(5)  # 25

# Multi-argomento
somma = lambda a, b: a + b

# Con condizionale
max = lambda a, b: a if a > b else b
```

## map — trasforma ogni elemento

```python
numeri = [1, 2, 3, 4, 5]

# Con lambda
quadrati = list(map(lambda x: x**2, numeri))  # [1, 4, 9, 16, 25]

# Con funzione esistente
str_numeri = list(map(str, numeri))  # ["1", "2", "3", "4", "5"]

# Multipla iterabile
a = [1, 2, 3]
b = [4, 5, 6]
somme = list(map(lambda x, y: x + y, a, b))  # [5, 7, 9]

# Alternativa: list comprehension (spesso più leggibile)
quadrati = [x**2 for x in numeri]
```

## filter — seleziona elementi

```python
numeri = [1, 2, 3, 4, 5]

pari = list(filter(lambda x: x % 2 == 0, numeri))  # [2, 4]

# Con None: rimuove valori falsy
valori = [0, 1, "", "ciao", None, [], [1]]
puliti = list(filter(None, valori))  # [1, "ciao", [1]]

# Alternativa: list comprehension
pari = [x for x in numeri if x % 2 == 0]
```

## reduce — riduce a singolo valore

Da `functools` (non built-in):

```python
from functools import reduce

numeri = [1, 2, 3, 4, 5]

# Somma cumulativa
somma = reduce(lambda a, b: a + b, numeri)  # 15
# Equivalente: ((((1+2)+3)+4)+5)

# Con valore iniziale
prodotto = reduce(lambda a, b: a * b, numeri, 1)  # 120

# Esempio: massimo
massimo = reduce(lambda a, b: a if a > b else b, numeri)  # 5

# Alternativa moderna: sum(), all(), any(), max(), min()
```

## Quando usare cosa

| Funzione | Cosa fa | Alternativa moderna |
|---|---|---|
| `map(func, seq)` | Trasforma elementi | List comprehension |
| `filter(func, seq)` | Seleziona elementi | List comprehension con if |
| `reduce(func, seq)` | Riduce a singolo valore | Loop esplicito o sum() |
| `lambda x: expr` | Funzione anonima | Funzione definita con def |
