---
topic: "Pattern Funzionali Avanzati — Python"
tags: [python, functional, advanced, pipeline, currying]
nav_prev: "[[functools.md]]"
---
Riferimento ufficiale: [docs.python.org/3/howto/functional.html](https://docs.python.org/3/howto/functional.html)

Pattern funzionali avanzati che Python supporta parzialmente o con librerie helper:

- **Currying**: trasformazione `f(a, b, c)` → `f(a)(b)(c)` via `functools.partial`
- **Pipeline**: composizione sequenziale di funzioni (operator `|` style)
- **Composition**: `f ∘ g` = `f(g(x))` con funzioni
- **Lazy evaluation**: generatori infiniti che producono valori on-demand
- **Trampoline**: ricorsione safe senza stack overflow

Vedi anche: [[BE-NOTES/Python/Funzionale/Concetti Base|Base FP]], [[BE-NOTES/Python/Tecnologie/Generator e Iterator|Generator]], [[BE-NOTES/Python/Funzionale/itertools|itertools]] per lazy evaluation.

Convertire una funzione multi-argomento in una catena di funzioni
ad un solo argomento:

```python
from functools import partial, wraps

def curry(func):
    """Decoratore che curryficizza una funzione."""
    @wraps(func)
    def curried(*args, **kwargs):
        if len(args) + len(kwargs) >= func.__code__.co_argcount:
            return func(*args, **kwargs)
        return curry(partial(func, *args, **kwargs))
    return curried

@curry
def somma(a: int, b: int, c: int) -> int:
    return a + b + c

somma(1)(2)(3)  # 6
somma(1, 2)(3)  # 6
```

## Pipeline di funzioni

```python
from functools import reduce

def pipeline(valore_iniziale, *funzioni):
    """Applica funzioni in sequenza: f(g(h(x)))."""
    return reduce(lambda v, f: f(v), funzioni, valore_iniziale)

# Uso
risultato = pipeline(
    "  Ciao Mondo!  ",
    str.strip,
    str.upper,
    lambda s: s.replace(" ", "_"),
    lambda s: s + "!!!"
)
print(risultato)  # "CIAO_MONDO_!!!"

# Alternativa: pipe operator-like con classi
class Pipe:
    def __init__(self, valore):
        self.valore = valore
    def __or__(self, func):
        return Pipe(func(self.valore))
    def __call__(self):
        return self.valore

result = Pipe(" Ciao ") | str.strip | str.upper | len
result()  # 4
```

## Function composition con decoratori

```python
def componi(*funzioni):
    """Composizione inversa: componi(f, g)(x) == f(g(x))."""
    def applica(valore):
        for f in reversed(funzioni):
            valore = f(valore)
        return valore
    return applica

# Oppure con reduce
def componi(*funzioni):
    from functools import reduce
    return reduce(lambda f, g: lambda x: f(g(x)), funzioni)

maiuscolo_esclama = componi(str.upper, lambda s: s + "!")
maiuscolo_esclama("ciao")  # "CIAO!"
```

## Lazy evaluation con generatori

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Prende solo ciò che serve
fib = fibonacci()
primi_10 = [next(fib) for _ in range(10)]

# Con itertools
from itertools import islice
primi_20 = list(islice(fibonacci(), 20))
```

## Trampoline (ricorsione senza stack overflow)

```python
def trampoline(func):
    """Converte ricorsione in iterazione."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        while callable(result):
            result = result()
        return result
    return wrapper

@trampoline
def fattoriale(n: int, acc: int = 1):
    if n == 0:
        return acc
    return lambda: fattoriale(n - 1, n * acc)
```
