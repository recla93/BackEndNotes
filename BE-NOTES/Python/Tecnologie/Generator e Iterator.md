---
topic: "Generator e Iterator — Python"
tags: [python, generator, iterator, lazy]
nav_prev: "[[Context Manager.md]]"
nav_next: "[[Async e AsyncIO.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/classes.html#iterators](https://docs.python.org/3/tutorial/classes.html#iterators)

Ogni `for` loop in Python usa il **protocollo iteratore**: `__iter__()` → `__next__()` → `StopIteration`. I **generator** sono il modo più semplice per creare iteratori: una funzione con `yield` invece di `return`.

Differenza chiave:
- `return` produce **un valore** e termina
- `yield` produce **un valore** e sospende, riprendendo alla chiamata successiva
- I generator sono **lazy**: calcolano valori on-demand, risparmiando memoria

Vedi anche: [[BE-NOTES/Python/Tecnologie/Async e AsyncIO|AsyncIO]] (le coroutine usano `yield` internamente), [[BE-NOTES/Python/Funzionale/itertools|itertools]] (lazy iterator library).

Ogni oggetto iterabile implementa `__iter__` e `__next__`:

```python
numeri = [1, 2, 3]
it = iter(numeri)     # chiama numeri.__iter__()
next(it)  # 1         # chiama it.__next__()
next(it)  # 2
next(it)  # 3
next(it)  # StopIteration
```

### ⚠️ Iteratori esauribili

Un iteratore si consuma una volta sola. Dopo aver iterato, è esaurito.
```python
numeri = [1, 2, 3]
it = iter(numeri)
list(it)  # [1, 2, 3]
list(it)  # [] — esaurito!
```
Una lista è **iterabile** ma non **iteratore**: puoi ottenere `iter(lista)` quante volte vuoi. Un generatore è entrambi, ma si consuma.

### `for` loop = syntactic sugar per iteratore

```python
for x in [1, 2, 3]:
    print(x)

# È equivalente a:
it = iter([1, 2, 3])
while True:
    try:
        x = next(it)
        print(x)
    except StopIteration:
        break
```

## Generatore — funzione che usa yield

**Come funziona**: quando Python incontra `yield` in una funzione, non la esegue subito — restituisce un oggetto generatore. Ogni chiamata a `next(gen)` esegue fino al prossimo `yield`, congela lo stack frame (variabili locali, puntatore d'istruzione) e restituisce il valore. Alla chiamata successiva, riprende da dove si era fermato.

La differenza con `return`: `return` termina la funzione e distrugge lo stack frame. `yield` sospende e preserva lo stato. Per questo serve `next()` per "svegliare" il generatore.

```python
def conto_fino_a(n: int):
    """Generatore: mantiene stato tra le chiamate."""
    i = 1
    while i <= n:
        yield i
        i += 1

gen = conto_fino_a(3)
next(gen)  # 1
next(gen)  # 2
next(gen)  # 3
next(gen)  # StopIteration

# I generatori sono lazy: calcolano un valore per volta
```

## Yield da / generator delegation

```python
def generator_principale():
    yield "primo"
    yield from sotto_generatore()  # delega a altro generatore
    yield "ultimo"

def sotto_generatore():
    yield "a"
    yield "b"
    yield "c"

list(generator_principale())  # ["primo", "a", "b", "c", "ultimo"]
```

## Generator expression

```python
# Come list comprehension ma lazy
import sys

quadrati_lista = [x**2 for x in range(1000)]  # lista: occupa memoria
quadrati_gen = (x**2 for x in range(1000))    # generatore: lazy

sys.getsizeof(quadrati_lista)  # ~8856 bytes
sys.getsizeof(quadrati_gen)    # ~200 bytes

# Utile per pipeline lazy
numeri = range(100)
pipeline = (x**2 for x in numeri if x % 2 == 0)
```

## Send, throw, close — generatori bidirezionali

```python
def logger():
    """Generatore che accetta input via .send()."""
    print("Logger avviato")
    while True:
        msg = yield  # riceve messaggio
        print(f"[LOG] {msg}")

gen = logger()
next(gen)          # avvia fino al primo yield
gen.send("Ciao")   # [LOG] Ciao
gen.send("Come va?")  # [LOG] Come va?
gen.close()        # StopIteration

# Alternativa: @asynccontextmanager per contesto async
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Generatore esaurito al secondo uso | Iteratori si consumano una volta sola | Ricrea il generatore o converti in lista se serve ri-leggerlo |
| `StopIteration` non catturata | `next(gen)` senza default né try/except | Usa `next(gen, default)` o un `for` loop |
| `return` con valore in generatore | `return val` in funzione con `yield` termina il generatore e mette il valore in `StopIteration.value` | Usa `yield from` o solo `return` senza valore |
| Pipeline lazy non eseguita | Generatore mai consumato — nessun codice eseguito | Itera il generatore (es. `list()`, `for`) per eseguirlo |
| `RecursionError` con ricorsione infinita | Generatore ricorsivo senza condizione di terminazione | Aggiungi limite di profondità o usa iterazione |


## Iteratore personalizzato

```python
class Fibonacci:
    def __init__(self, limite: int):
        self.limite = limite
        self.a, self.b = 0, 1
        self.conteggio = 0

    def __iter__(self):
        return self  # l'iteratore è sé stesso

    def __next__(self):
        if self.conteggio >= self.limite:
            raise StopIteration
        self.conteggio += 1
        self.a, self.b = self.b, self.a + self.b
        return self.a

for n in Fibonacci(10):
    print(n)
```
