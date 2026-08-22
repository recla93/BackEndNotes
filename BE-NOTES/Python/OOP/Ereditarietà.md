---
topic: "Ereditarietà — Python OOP"
tags: [python, oop, inheritance, abstract, protocol]
---
nav_prev: "[[Incapsulamento e Property.md]]"
nav_next: "[[Magic Methods.md]]"

Riferimento ufficiale: [docs.python.org/3/tutorial/classes.html#inheritance](https://docs.python.org/3/tutorial/classes.html#inheritance)

Python supporta ereditarietà **singola**, **multipla** e **ABC** (Abstract Base Classes). A differenza di Java:
- Niente `extends` / `implements` — si eredita con `(Genitore)` e basta
- Ereditarietà multipla con MRO (C3 linearization)
- `super()` segue il MRO, non il genitore diretto
- `abc.ABC` + `@abstractmethod` per classi astratte (come `abstract class` in Java)
- `Protocol` per duck typing strutturale (come interfacce implicite)

Vedi anche: [[BE-NOTES/Python/OOP/Classi e Oggetti|Classi e Oggetti]], [[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]] per Protocol.

## Ereditarietà singola

```python
class Animale:
    def __init__(self, nome: str):
        self.nome = nome

    def verso(self) -> str:
        return "..."

class Cane(Animale):
    def verso(self) -> str:
        return "Bau!"

    def scodinzola(self) -> str:
        return f"{self.nome} scodinzola"

c = Cane("Fido")
c.verso()        # "Bau!" (override)
c.scodinzola()   # "Fido scodinzola"
isinstance(c, Animale)  # True
```

## super()

```python
class Gatto(Animale):
    def __init__(self, nome: str, colore: str):
        super().__init__(nome)  # chiama costruttore genitore
        self.colore = colore

    def verso(self) -> str:
        return super().verso() + " ... Miao!"  # estende
```

## Ereditarietà multipla

```python
class Volante:
    def vola(self) -> str:
        return "Sto volando"

class Nuotante:
    def nuota(self) -> str:
        return "Sto nuotando"

class Anatra(Volante, Nuotante):
    pass

a = Anatra()
a.vola()    # "Sto volando"
a.nuota()   # "Sto nuotando"
```

## MRO (Method Resolution Order)

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
# Python usa C3 linearization (algoritmo di ordinamento)
```

### `super()` e il MRO — attenzione!

A differenza di Java dove `super` punta sempre al genitore diretto, in Python `super()` segue il **MRO**. Questo è fondamentale nell'ereditarietà multipla: `super()` chiama il *prossimo* nella catena MRO, non necessariamente il genitore dichiarato.

```python
class A:
    def metodo(self): return "A"

class B(A):
    def metodo(self): return f"B -> {super().metodo()}"  # chiama A

class C(A):
    def metodo(self): return f"C -> {super().metodo()}"  # chiama A

class D(B, C):
    def metodo(self): return f"D -> {super().metodo()}"  # chiama B (non C!)

# MRO di D: D -> B -> C -> A -> object
# Quindi super() in B chiama C (non A!)
# Questo si chiama "cooperative multiple inheritance"
```

**Senza cooperative inheritance**, i metodi di C verrebbero saltati se B chiama direttamente `A.metodo(self)` invece di `super().metodo()`. Per questo nei diamond problem Python è più sicuro: `super()` garantisce che ogni classe nella gerarchia venga chiamata una e una sola volta.

## Abstract Base Classes

```python
from abc import ABC, abstractmethod

class Veicolo(ABC):
    @abstractmethod
    def muovi(self) -> str:
        """Le sottoclassi DEVONO implementare."""
        ...

    def descrizione(self) -> str:
        """Metodo concreto — disponibile per tutte."""
        return "Sono un veicolo"

class Auto(Veicolo):
    def muovi(self) -> str:
        return "L'auto si muove su strada"

class Barca(Veicolo):
    def muovi(self) -> str:
        return "La barca naviga"

v = Auto()  # OK
v = Veicolo()  # TypeError (non istanziabile)
```

## Protocol (duck typing strutturale, Python 3.8+)

```python
from typing import Protocol

class Volabile(Protocol):
    def vola(self) -> str: ...

class Aereo:
    def vola(self) -> str:
        return "Decollo!"

# Duck typing: Aereo è compatibile con Volabile senza ereditare
def fai_volare(cosa: Volabile) -> str:
    return cosa.vola()

fai_volare(Aereo())  # OK
```

## Best practice

- **Preferisci composizione a ereditarietà** (più flessibile, meno accoppiamento)
- **ABC per interfacce formali e documentazione**
- **Protocol per duck typing** — non richiede gerarchia di classi
- **Ereditarietà multipla è potente ma complessa** — documenta il MRO
- **`super().__init__()` sempre in MRO-aware classes** (anche con diamond problem)
- **Non ereditare da classi concrete per riuso** usa composizione

## Security

- **ABC previene istanziazione accidentale** — utile per garantire implementazione
- **Protocol non runtime-safe** — mypy/pyright catturano errori, non Python runtime
- **Override di metodi può rompere LSP** — segui il principio di sostituzione di Liskov
