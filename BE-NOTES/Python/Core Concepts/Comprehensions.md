---
topic: "Comprehensions — Python"
tags: [python, base, comprehension, list, dict, set]
nav_prev: "[[Liste Tuple Set Dict.md]]"
nav_next: "[[Funzioni.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/datastructures.html#list-comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)

Le **comprehension** sono un costrutto sintattico che crea una nuova sequenza applicando una trasformazione con filtro opzionale, il tutto in una singola riga leggibile. Sono l'alternativa Pythonica a `map()` e `filter()` nella maggior parte dei casi.

Esistono quattro varianti: `list comprehension`, `dict comprehension`, `set comprehension` e `generator expression` (parentesi tonde).

Una comprehension si scrive come `[espressione for variabile in iterabile if condizione]`. La parte `if condizione` è opzionale. L'ordine delle operazioni segue il nesting logico: prima cicla, poi filtra, poi trasforma.

Vedi anche:
[[BE-NOTES/Python/Core Concepts/Liste Tuple Set Dict|Liste Tuple Set Dict]],
[[BE-NOTES/Python/Funzionale/Lambda map filter reduce|Lambda map filter reduce]],
[[BE-NOTES/Python/Funzionale/itertools|itertools]].

## List comprehension

```python
numeri = [1, 2, 3, 4, 5]

# Trasformazione: [espr. for var in iterabile]
quadrati = [n**2 for n in numeri]          # [1, 4, 9, 16, 25]

# Con filtro: [espr. for var in iterabile if condizione]
pari = [n for n in numeri if n % 2 == 0]   # [2, 4]

# Trasformazione + filtro
pari_quadrati = [n**2 for n in numeri if n % 2 == 0]  # [4, 16]

# Doppio ciclo (prodotto cartesiano)
coppie = [(a, b) for a in [1, 2] for b in ['x', 'y']]
# [(1, 'x'), (1, 'y'), (2, 'x'), (2, 'y')]
```

La comprehension crea una nuova lista in memoria. Per iterazioni grandi (milioni di elementi) preferisci un **generatore**. L'ordine dei `for` annidati segue lo stesso ordine di un ciclo `for` classico.

## Dict comprehension

```python
parole = ["ciao", "mondo", "python"]

# {chiave: valore for var in iterabile}
lunghezze = {parola: len(parola) for parola in parole}
# {'ciao': 4, 'mondo': 5, 'python': 6}

# Con filtro
parole_lunghe = {p: len(p) for p in parole if len(p) > 4}
# {'mondo': 5, 'python': 6}

# Invertire un dict
originale = {'a': 1, 'b': 2}
invertito = {v: k for k, v in originale.items()}
# {1: 'a', 2: 'b'}
```

Attenzione a chiavi duplicate: l'ultima vince. Se la funzione di trasformazione è costosa, valutala una sola volta.

## Set comprehension

```python
numeri = [1, 2, 2, 3, 3, 3]

# {espr. for var in iterabile}
unici = {n for n in numeri}                # {1, 2, 3}
pari_unici = {n for n in numeri if n % 2 == 0}  # {2}

# Utile per deduplicare con trasformazione
testi = ["Ciao", "ciao", "MONDO"]
normalizzati = {t.lower() for t in testi}  # {'ciao', 'mondo'}
```

Il set comprehension è identico alla dict comprehension ma senza i `:`.

## Generator expression

```python
numeri = [1, 2, 3, 4, 5]

# (espr. for var in iterabile) — lazy!
gen = (n**2 for n in numeri)
print(list(gen))  # [1, 4, 9, 16, 25]

# Se unico argomento di funzione, le parentesi tonde si omettono
somma = sum(n**2 for n in numeri)          # 55
any(n > 3 for n in numeri)                 # True
all(n > 0 for n in numeri)                 # True
```

Il generator è **lazy**: non occupa memoria per tutta la sequenza. Perfetto per pipeline su dati grandi e per funzioni `sum()`, `any()`, `all()`.

## Comprehension vs map/filter

```python
numeri = [1, 2, 3, 4, 5]

# map + lambda vs comprehension
list(map(lambda n: n**2, numeri))          # OK
[n**2 for n in numeri]                     # Pythonico

# filter + map vs comprehension
list(map(lambda n: n**2, filter(lambda n: n % 2 == 0, numeri)))  # Leggibilità?
[n**2 for n in numeri if n % 2 == 0]       # Vincitore
```

La comprehension vince in leggibilità quando la logica è semplice. `map()`/`filter()` hanno senso solo se hai già una funzione definita (es. `map(str.upper, lista)`).

## Errori comuni

- **Comprehension troppo lunga**: se ha 3+ `for` annidati, scrivi un ciclo normale. La leggibilità viene prima.
- **Dimenticare che crea una nuova lista**: per milioni di elementi, usa generator expression.
- **Usare `[]` quando serve `{}`**: `{x for x in ...}` è set, `{k: v for ...}` è dict, `[x for ...]` è list.
- **Modificare l'iterabile mentre lo si itera**: non farlo mai, in nessun costrutto.
- **Confondere ordine dei `for`**: `[expr for a in A for b in B]` equivale a `for a in A: for b in B`.

## Best Practices & Conventions

- Preferisci comprehension a `map()`/`filter()` per operazioni semplici e leggibili.
- Mantieni la comprehension entro ~80 caratteri. Se è più lunga, spezzala in più righe o usa un ciclo.
- Usa generator expression per pipeline su dati grandi o come argomento di funzione aggregante.
- Non nidificare comprehension oltre 2 livelli. Oltre, scrivi cicli espliciti.
- Per creare dict da due liste allineate: `{k: v for k, v in zip(chiavi, valori)}`.
- La condizione `if` va **dopo** il `for`, non prima.
