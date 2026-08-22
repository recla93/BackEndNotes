---
topic: "Liste, Tuple, Set, Dict — Python"
tags: [python, base, collections, data-structures]
nav_prev: "[[Strutture di Controllo.md]]"
nav_next: "[[File e IO.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/datastructures.html](https://docs.python.org/3/tutorial/datastructures.html)

Python offre 4 strutture dati fondamentali integrate: `list` (array mutabile), `tuple` (sequenza immutabile), `set` (insieme senza duplicati), `dict` (mappa chiave-valore).

La scelta della struttura giusta impatta leggibilità, performance e sicurezza. Regola generale:
- **Ordinato e mutabile** → `list`
- **Ordinato e immutabile** → `tuple`
- **Unicità e operazioni insiemistiche** → `set`
- **Associazioni key-value** → `dict`

Vedi anche: [[BE-NOTES/Python/Core Concepts/Variabili e Tipi|Variabili e Tipi]], [[BE-NOTES/Python/Funzionale/itertools|itertools]] per operazioni avanzate su collezioni.

## Liste — `list`

Equivalente di `ArrayList<E>` in Java, ma con sintassi letterale e più metodi built-in.

```python
numeri = [1, 2, 3, 4, 5]

# Accesso
numeri[0]     # 1
numeri[-1]    # 5 (ultimo)
numeri[1:3]   # [2, 3] (slicing, stop escluso)

# Modifica
numeri.append(6)       # [1,2,3,4,5,6]
numeri.insert(0, 0)    # [0,1,2,3,4,5,6]
numeri.extend([7, 8])  # [0,1,2,3,4,5,6,7,8]
numeri.pop()           # rimuove e restituisce ultimo
numeri.remove(3)       # rimuove primo match di 3
numeri.sort()          # ordina in-place
numeri.reverse()       # inverte in-place

# Ricerca
numeri.index(3)  # indice del primo 3
numeri.count(2)  # quante volte appare 2
3 in numeri      # True/False

# List comprehension — alternativa Pythonica a stream().filter().map()
pari = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]
quadrati = [x**2 for x in range(10)]          # [0, 1, 4, 9, ...]
```

### Slicing — estrarre sottosequenze
Come Java `subList()` ma più potente: `[start:stop:step]`:
```python
numeri = [0, 1, 2, 3, 4, 5]
numeri[1:4]     # [1, 2, 3] — da 1 a 3 (stop escluso)
numeri[:3]      # [0, 1, 2] — dall'inizio
numeri[3:]      # [3, 4, 5] — fino alla fine
numeri[::2]     # [0, 2, 4] — step 2
numeri[::-1]    # [5, 4, 3, 2, 1, 0] — rovescia
```

### Copia: shallow vs deep
```python
originale = [1, [2, 3], 4]
copia = originale.copy()       # shallow: lista interna è condivisa
copia[1][0] = 99               # modifica ANCHE originale!

import copy
copia_deep = copy.deepcopy(originale)  # tutto indipendente
```

## Tuple — `tuple`

Come `List.of(...)` in Java — immutabile. Più leggera di una lista (meno overhead di memoria). L'immutabilità la rende **hashable**, quindi usabile come chiave dict.

```python
coordinate = (10, 20)
singleton = (1,)  # virgola necessaria — senza, è int!

# Unpacking — come Java record pattern
x, y = coordinate  # x=10, y=20
x, *resto = (1, 2, 3, 4)  # x=1, resto=[2,3,4] — unpacking esteso

# Usi tipici
# - Valori di ritorno multipli (Java non lo permette)
# - Chiavi di dict (immutabile)
# - Dati che non devono cambiare (sicurezza)
```

### Named tuple — tuple con nomi (alternativa leggera a classi)
```python
from collections import namedtuple
Punto = namedtuple("Punto", ["x", "y"])
p = Punto(10, 20)
p.x       # 10 — accesso per nome
p[0]      # 10 — accesso per indice (tuple compatibile)
x, y = p  # unpacking
```

## Set — `set`

Non ordinato, valori unici, mutabile.

```python
colori = {"rosso", "blu", "verde"}
colori.add("giallo")
colori.discard("blu")  # rimuove senza errore se non esiste

# Operazioni di insieme
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b  # unione:     {1,2,3,4,5,6}
a & b  # intersezione: {3,4}
a - b  # differenza:  {1,2}
a ^ b  # differenza simmetrica: {1,2,5,6}

# Set comprehension
pari_set = {x for x in range(10) if x % 2 == 0}
```

## Dizionari — `dict`

Equivalente di `HashMap<K, V>` in Java. Key-value, ordinato per inserimento (da Python 3.7+, prima era non ordinato).

```python
persona = {
    "nome": "Mario",
    "eta": 25,
    "citta": "Roma"
}

# Accesso sicuro (Java: getOrDefault)
persona["nome"]                           # "Mario" — KeyError se manca
persona.get("telefono")                   # None (non errore — Java-style)
persona.get("telefono", "non specificato")  # default
persona.setdefault("telefono", "000")     # se manca, setta e restituisce default

# Modifica
persona["eta"] = 26
persona.update({"citta": "Milano", "lavoro": "Dev"})  # merge
del persona["telefono"]
persona.pop("eta")

# Iterazione — Java: entrySet(), keySet(), values()
for key in persona:                           # chiavi (come keySet())
for key, val in persona.items():              # key-value (come entrySet())
for val in persona.values():                  # valori (come values())

# Dict comprehension (Java: Collectors.toMap)
quadrati_dict = {x: x**2 for x in range(5)}  # {0:0, 1:1, 2:4, 3:9, 4:16}

# Merge dict (Python 3.9+)
a = {"x": 1, "y": 2}
b = {"y": 3, "z": 4}
c = a | b  # {"x": 1, "y": 3, "z": 4} — override y, aggiunge z
```

## Quando usare cosa

| Tipo | Mutabile | Ordinato | Quando usare |
|------|----------|---------|-------------|
| `list` | ✅ | ✅ | Collezione modificabile, accesso per indice |
| `tuple` | ❌ | ✅ | Dati immutabili, chiavi dict, unpacking |
| `set` | ✅ | ❌ | Unicità, operazioni di insieme |
| `dict` | ✅ | ✅ (3.7+) | Mappature, cache, lookup veloci |
| `frozenset` | ❌ | ❌ | Set immutabile, usabile come chiave dict |
