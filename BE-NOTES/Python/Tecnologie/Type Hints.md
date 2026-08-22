---
topic: "Type Hints — Python"
tags: [python, types, mypy, typing]
nav_next: "[[Context Manager.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html)

## Perché

Introdotti in Python 3.5 (PEP 484), i type hints sono **opzionali** ma diventati standard in ogni progetto serio. Vantaggi:
- **Documentazione eseguibile** verificata da mypy/pyright
- **Autocompletamento IDE** preciso
- **Catch bug** prima dell'esecuzione
- **Refactoring sicuro** (IDE rinomina con confidence)
- **Zero overhead runtime** — vengono ignorate dall'interprete

### Type Hints non sono tipizzazione statica
A differenza di Java/C# dove il tipo è **vincolante** (se dichiari `int x`, non puoi assegnargli una stringa), i type hints Python sono **solo annotazioni**:
```python
x: int = 5
x = "ciao"  # Python NON lancia errore! Solo mypy/pyright lo segnalano
```
Sono un aiuto per lo sviluppatore e gli strumenti di analisi, non per l'interprete.

## Tipi base

```python
nome: str = "Mario"
eta: int = 25
altezza: float = 1.85
attivo: bool = True
valore: None = None

# Collezioni (Python 3.9+)
numeri: list[int] = [1, 2, 3]
coppia: tuple[str, int] = ("Mario", 25)
opzioni: set[str] = {"a", "b"}
dati: dict[str, int] = {"chiave": 42}
```

## Union e Optional

```python
# Union (Python 3.10+)
valore: int | str | None = None

# Optional: str | None (equivalente)
from typing import Optional
indirizzo: Optional[str] = None  # str | None

# any — qualsiasi tipo
from typing import Any
libero: Any = "qualunque"
libero = 42  # ok
```

## Callable — firme di funzioni come tipo

```python
from typing import Callable

# Callable[[tipi_argomenti], tipo_ritorno]
def esegui(func: Callable[[int, str], bool], n: int, s: str) -> bool:
    return func(n, s)
```
### Cosa significa "firme avanzate"
"Firme avanzate" significa che con i type hints puoi descrivere **strutture di tipi complesse**: non solo `int`, `str`, ma funzioni che ricevono funzioni (callback), classi che lavorano su tipi generici, funzioni con più firme possibili (overload).

## Generics — spiegazione

I Generics permettono di scrivere funzioni/classi che lavorano con **un tipo senza specificarlo**, mantenendo però il type checking:

```python
from typing import TypeVar, Generic

T = TypeVar("T")  # "un tipo qualsiasi, ma consistente"

class Contenitore(Generic[T]):  # Contenitore di T
    def __init__(self, valore: T):
        self.valore = valore
    def ottieni(self) -> T:
        return self.valore

# Quando lo usi, il tipo viene fissato:
contenitore_int = Contenitore[int](42)
valore: int = contenitore_int.ottieni()  # mypy sa che è int

contenitore_str = Contenitore[str]("ciao")
valore: str = contenitore_str.ottieni()  # mypy sa che è str
```
Senza Generics, dovresti usare `Any` e perdere il type checking, o fare una classe per ogni tipo.

## Protocol (duck typing strutturale) — spiegazione

Python 3.8+, alternativa alle ABC. L'idea: **se un oggetto ha il metodo `vola()`, allora è Volabile** — non serve che erediti da una classe base:

```python
from typing import Protocol

class Volabile(Protocol):
    def vola(self) -> str: ...  # definisco il "contratto"

def fai_volare(cosa: Volabile) -> str:
    return cosa.vola()

class Aereo:
    def vola(self) -> str:
        return "Decollo!"

class Uccello:
    def vola(self) -> str:
        return "Svolazzo!"

# Entrambi funzionano senza ereditare da Volabile
fai_volare(Aereo())     # OK
fai_volare(Uccello())   # OK
```
A differenza di Java (interfacce esplicite) o delle ABC (che richiedono `extends`), Protocol segue il **duck typing**: "se cammina come un'anatra e starnazza come un'anatra, allora è un'anatra". MyPy/Pyright verificano la compatibilità strutturale.

## Overload — spiegazione

Permette di dichiarare **più firme per la stessa funzione** — come l'overloading in Java/C#:

```python
from typing import overload

@overload
def processa(valore: int) -> int: ...

@overload
def processa(valore: str) -> str: ...

@overload
def processa(valore: list[str]) -> list[int]: ...

def processa(valore: int | str | list[str]) -> int | str | list[int]:
    if isinstance(valore, int):
        return valore * 2
    if isinstance(valore, str):
        return valore.upper()
    return [len(s) for s in valore]
```

## Literal, Final, TypedDict, NotRequired — spiegazioni

### Literal — "solo questi valori"
```python
from typing import Literal

def set_mode(mode: Literal["read", "write", "append"]) -> None:
    # mode può essere SOLO "read", "write" o "append"
    ...

set_mode("read")    # OK
set_mode("delete")  # mypy: errore!
```
Utile per: flag, modalità, costanti con set limitato di valori.

### Final — non riassegnabile
```python
from typing import Final

PI: Final = 3.14159
PI = 3.0  # mypy: errore! Final non può essere riassegnato
```
Utile per: costanti, configurazioni che non devono cambiare.

### TypedDict — dict con struttura fissa
```python
from typing import TypedDict, NotRequired

class Utente(TypedDict):
    nome: str
    eta: int
    email: NotRequired[str]  # può mancare

# Vantaggio: mypy controlla le chiavi e i tipi
utente: Utente = {"nome": "Mario", "eta": 25}  # OK
utente: Utente = {"nome": "Mario"}              # errore: manca eta
```
Utile per: dati JSON non validati, kwargs strutturati, risposte API dinamiche.

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `TypeError: 'type' object is not subscriptable` | `list[int]`, `dict[str, int]` usato con Python <3.9 | Usa `from __future__ import annotations` o typing `List[int]`, `Dict[str, int]` |
| mypy segnala errore ma runtime funziona | Type hint non corrisponde al valore reale | Il type hint è un'annotazione — l'errore è solo di mypy, non blocca il runtime |
| `Optional[str]` vs `str \| None` confusione | Sono equivalenti — ma `Optional[str]` non significa "opzionale nel senso di default None" | `Optional[str] = None` è corretto, `Optional[str] = "default"` è sbagliato semanticamente |
| `TypeVar` senza vincoli accetta `Any` | `T = TypeVar("T")` senza `bound=` non limita il tipo | Usa `bound=SomeClass` o vincoli con `TypeVar("T", int, str)` |
| `Protocol` non rileva metodi mancanti | Il tipo passato ha lo stesso nome ma firma diversa | MyPy controlla la firma esatta — non solo il nome del metodo |
| `Any` usato come stampella | Troppi `Any` annullano il type checking | Usa tipi precisi o `object` se devi essere generico |

## Tools di type checking

- **MyPy**: `mypy src/` — il più maturo
- **Pyright** / **Pylance**: basato su TypeScript, usato da VSCode
- **Pyre**: di Meta
- **Pytype**: di Google
