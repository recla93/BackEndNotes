---
topic: "Funzioni — Python"
tags: [python, base, functions, scope]
nav_prev: "[[Variabili e Tipi.md]]"
nav_next: "[[Strutture di Controllo.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/controlflow.html#defining-functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)

In Python le funzioni sono **oggetti di prima classe**: possono essere assegnate a variabili, passate come argomenti, restituite da altre funzioni. Non c'è `public static` — si definiscono con `def` e basta.

A differenza di Java, Python supporta:
- Parametri con default e keyword arguments
- `*args` / `**kwargs` per numero variabile
- `lambda` per funzioni anonime
- Closure e scope annidato (LEGB)

Vedi anche: [[BE-NOTES/Python/Funzionale/Concetti Base|Funzionale]] per higher-order functions, [[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]] per firme avanzate.

## Definizione e parametri — spiegazioni

### Named arguments (keyword arguments)
In Python puoi passare argomenti **per nome**, non solo per posizione:
```python
# Posizionale: l'ordine conta
crea_profilo("Mario", 25, "Roma")

# Named: l'ordine NON conta, più leggibile
crea_profilo(nome="Mario", città="Roma", eta=25)
```
Perché usarli:
- **Chiarezza**: chi legge capisce cosa rappresenta ogni argomento
- **Ordine libero**: non devi ricordare la sequenza dei parametri
- **Skippare default**: puoi omettere parametri con default in mezzo

### Parametri di default e mutabilità
I parametri di default sono valutati **una volta sola** (alla definizione). Attenzione ai mutabili:
```python
def sbagliato(x, lista=[]):  # ← Questo [] è lo STESSO oggetto ogni volta
    lista.append(x)
    return lista

sbagliato(1)  # [1]
sbagliato(2)  # [1, 2] — non è una nuova lista!
```
Correzione: usa `None` e crea la lista dentro:
```python
def corretto(x, lista=None):
    if lista is None:
        lista = []
    lista.append(x)
    return lista
```

## *args e **kwargs — spiegazioni

### `*args`: numero variabile di argomenti posizionali
Il `*` raccoglie gli argomenti extra in una **tupla**:
```python
def log(*messaggi):
    for msg in messaggi:
        print(f"[LOG] {msg}")

log("errore")           # un argomento
log("warn", "debug")    # due argomenti
log()                   # zero argomenti

# Si può usare anche per "unpacking":
numeri = [1, 2, 3]
print(*numeri)  # stampa "1 2 3" (come se passassi 3 argomenti separati)
```
Dove si usa: decoratori ([[BE-NOTES/Python/OOP/Decoratori|vedi]]) che devono passare qualsiasi argomento alla funzione wrappata, wrapper, logger, `zip()`.

### `**kwargs`: numero variabile di keyword arguments
Il `**` raccoglie gli argomenti nominati extra in un **dict**:
```python
def configura(**opzioni):
    for chiave, valore in opzioni.items():
        print(f"{chiave} = {valore}")

configura(host="localhost", port=8080, debug=True)

# Utile per passare parametri a funzioni interne:
def chiamata_esterna(url, **params):
    requests.get(url, params=params)  # passa tutto come query params
```
Dove si usa: configurazioni flessibili, ereditarietà con `super()`, overloading di funzioni, `**dict_unpacking`.

### Combinazione
Ordine obbligatorio: `posizionali, *args, keyword-only, **kwargs`:
```python
def f(a, b, *args, c, **kwargs): ...
# a, b = posizionali obbligatori
# *args = extra posizionali (tupla)
# c = keyword-only (deve essere passato per nome)
# **kwargs = extra keyword (dict)
```

## Scope (LEGB) — spiegazione

### Il problema: dove cerca Python le variabili?
Quando usi un nome (es. `x`), Python cerca in 4 scopes, in questo ordine:

**L**ocal → **E**nclosing → **G**lobal → **B**uilt-in

```python
x = "globale"        # Global scope — visibile in tutto il file

def esterna():
    x = "enclosing"  # Enclosing scope — visibile in esterna e interna
    
    def interna():
        x = "locale"  # Local scope — visibile solo in interna
        return x
    return interna()
```
Se usi `x = 5` dentro una funzione, Python **crea una nuova variabile locale**, non modifica la globale. Per modificare una globale:
```python
contatore = 0

def incrementa():
    global contatore  # "dichiaro" che voglio modificare la globale
    contatore += 1    # ora modifica la globale, non ne crea una locale
```
Per modificare una variabile **enclosing** (da funzione interna a esterna):
```python
def esterna():
    x = 0
    def interna():
        nonlocal x    # come global, ma per l'enclosing scope
        x += 1
```

### Closure: quando una funzione "ricorda" il suo contesto
Una **closure** è una funzione interna che "cattura" variabili dello scope enclosing:
```python
def crea_moltiplicatore(fattore):
    def moltiplica(valore):
        return valore * fattore  # fattore viene dallo scope esterno!
    return moltiplica

per_2 = crea_moltiplicatore(2)   # closure: "ricorda" fattore=2
per_3 = crea_moltiplicatore(3)   # un'altra closure: "ricorda" fattore=3

per_2(5)  # 10
per_3(5)  # 15
```
Vantaggio: **factory di funzioni** — crei funzioni specializzate senza ripetere codice. Usato in: decoratori, callback, `functools.partial()`.

## Type Hints nelle funzioni — firme avanzate

I type hints possono descrivere firme complesse:

### Callable — funzioni come parametri
```python
from typing import Callable

# Callable[[tipi_argomenti], tipo_ritorno]
def esegui(
    callback: Callable[[int, str], bool],  # funzione che prende (int, str) → bool
    valore: int | None = None,
) -> str:
    if valore is None:
        return "nessun valore"
    return str(callback(valore, "test"))
```
Serve per: callback, handler, pipeline functions, strategy pattern.

### Firme avanzate (Callable, Generics, Overload)
Vedi [[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]] per:
- **Generics** — funzioni che lavorano su tipi generici (`T`)
- **Overload** — multiple firme per stessa funzione
- **Literal** — valori esatti come tipo
- **Protocol** — strutturale duck typing
- **ParamSpec** — preservare firme in decoratori (Python 3.10+)
