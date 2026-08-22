---
topic: "Decoratori — Python"
tags: [python, decorators, metaprogramming]
nav_prev: "[[Magic Methods.md]]"
---
Riferimento ufficiale: [docs.python.org/3/reference/compound_stmts.html#function](https://docs.python.org/3/reference/compound_stmts.html#function)

## Cosa sono

Un decoratore è una funzione che prende una funzione (o classe) e
restituisce una versione modificata, senza alterarne il codice sorgente.

Python li usa estensivamente: `@property`, `@classmethod`, `@staticmethod`,
`@dataclass`, `@lru_cache` sono tutti decoratori built-in.
Framework come FastAPI li usano per routing (`@app.get()`).

Vedi anche: [[BE-NOTES/Python/Funzionale/Concetti Base|Funzionale]] per higher-order functions, [[BE-NOTES/Python/OOP/Incapsulamento e Property|Property]] come esempio concreto.

## Decoratore semplice

```python
import functools
import time

def tempo(func):
    """Stampa quanto tempo impiega una funzione."""
    @functools.wraps(func)  # preserva nome, docstring, etc.
    def wrapper(*args, **kwargs):
        inizio = time.perf_counter()
        risultato = func(*args, **kwargs)
        fine = time.perf_counter()
        print(f"{func.__name__}: {fine - inizio:.3f}s")
        return risultato
    return wrapper

@tempo
def operazione_lenta():
    time.sleep(0.5)

operazione_lenta()  # stampa: "operazione_lenta: 0.500s"
```

## Decoratore con parametri

```python
def ripeti(n: int):
    """Esegue la funzione decorata n volte."""
    def decoratore(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                risultato = func(*args, **kwargs)
            return risultato
        return wrapper
    return decoratore

@ripeti(3)
def saluta(nome: str):
    print(f"Ciao {nome}")

saluta("Mario")  # stampa "Ciao Mario" 3 volte
```

## Decoratori built-in utili

```python
from dataclasses import dataclass
from functools import lru_cache, cached_property

# @property — già visto in property.md
# @classmethod, @staticmethod — già visti in classi.md
# @dataclass — genera __init__, __repr__, __eq__ automaticamente
@dataclass
class Punto:
    x: float
    y: float

# @lru_cache — memoizza risultati funzione
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

## Esempi pratici

```python
# ========== LOG ==========
import logging
logger = logging.getLogger(__name__)

def log_chiamata(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logger.info(f"Chiamo {func.__name__}(args={args}, kwargs={kwargs})")
        return func(*args, **kwargs)
    return wrapper

# ========== VALIDAZIONE ==========
def valida_positivo(func):
    @functools.wraps(func)
    def wrapper(valore: float):
        if valore <= 0:
            raise ValueError("Il valore deve essere positivo")
        return func(valore)
    return wrapper

# ========== RETRY ==========
import time

def retry(volte: int = 3, delay: float = 1.0):
    def decoratore(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for tentativo in range(volte):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if tentativo == volte - 1:
                        raise
                    print(f"Tentativo {tentativo+1} fallito: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decoratore
```
