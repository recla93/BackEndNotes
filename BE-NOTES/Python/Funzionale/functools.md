---
topic: "functools — Python"
tags: [python, functional, functools, caching]
nav_prev: "[[itertools.md]]"
nav_next: "[[Pattern Avanzati.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/functools.html](https://docs.python.org/3/library/functools.html)

Modulo built-in per funzioni higher-order e utilità funzionali. Contiene strumenti per:

- **Partial**: fissare argomenti (crea funzioni specializzate)
- **Cache**: `lru_cache`, `cached_property` per memoizzazione
- **Dispatch**: `singledispatch` per polimorfismo su tipo del primo argomento
- **Decorator utilities**: `wraps` per preservare metadata

Vedi anche: [[BE-NOTES/Python/OOP/Decoratori|Decoratori]], [[BE-NOTES/Python/Funzionale/Concetti Base|FP Concetti]], [[BE-NOTES/Python/Tecnologie/Async e AsyncIO|Async]] per caching in contesti concorrenti.

```python
from functools import partial

def potenza(base: float, esponente: float) -> float:
    return base ** esponente

# Crea versioni specializzate
quadrato = partial(potenza, esponente=2)
cubo = partial(potenza, esponente=3)

quadrato(5)  # 25
cubo(3)      # 27

# Utile con API che richiedono callback con firma fissa
def log(level: str, msg: str):
    print(f"[{level}] {msg}")

info = partial(log, "INFO")
error = partial(log, "ERROR")

info("Server avviato")   # [INFO] Server avviato
error("Errore fatale")   # [ERROR] Errore fatale
```

## wraps — preservare metadata

```python
from functools import wraps

def decoratore(func):
    @wraps(func)  # SENZA questo: nome = "wrapper"
    def wrapper(*args, **kwargs):
        """Wrapper interno."""
        return func(*args, **kwargs)
    return wrapper

@decoratore
def fai_qualcosa():
    """Documentazione originale."""
    pass

fai_qualcosa.__name__  # "fai_qualcosa" (invece di "wrapper")
```

## lru_cache — memoizzazione

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    """Calcola Fibonacci con cache automatica."""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# maxsize=None: cache illimitata
# typed=True: distingue int(3) e float(3.0)

fibonacci.cache_info()  # CacheInfo(hits=..., misses=..., maxsize=..., currsize=...)
fibonacci.cache_clear()  # svuota cache
```

## singledispatch — dispatch polimorfico

```python
from functools import singledispatch

@singledispatch
def formatta(valore) -> str:
    """Formatta un valore generico."""
    return str(valore)

@formatta.register(int)
def _(valore: int) -> str:
    return f"{valore:,}"

@formatta.register(float)
def _(valore: float) -> str:
    return f"{valore:.2f}"

@formatta.register(list)
def _(valore: list) -> str:
    return ", ".join(str(v) for v in valore)

formatta(1000)         # "1,000"
formatta(3.14159)      # "3.14"
formatta([1, 2, 3])    # "1, 2, 3"
```

## cached_property (Python 3.8+)

```python
from functools import cached_property

class Analisi:
    def __init__(self, dati: list[int]):
        self.dati = dati

    @cached_property
    def media(self) -> float:
        """Calcolato una sola volta, poi cache."""
        return sum(self.dati) / len(self.dati)
```
