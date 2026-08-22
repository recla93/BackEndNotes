---
topic: "Classi e Oggetti — Python OOP"
tags: [python, oop, classes, objects]
nav_next: "[[Incapsulamento e Property.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/classes.html](https://docs.python.org/3/tutorial/classes.html)

Python è **multi-paradigma**: supporta OOP ma non forza tutto in classi (come Java). A differenza di Java:
- **Niente `private`/`protected`** (solo convenzioni `_` e `__`)
- **`self` esplicito** (non `this` implicito)
- **Heritage multipla** (con MRO C3 linearization)
- **Duck typing** — se ha il metodo, funziona (vedi [[BE-NOTES/Python/OOP/Ereditarietà|Protocol]])
- **Tutto è un oggetto** (classi, funzioni, moduli)

Vedi anche: [[BE-NOTES/Python/OOP/Ereditarietà|Ereditarietà]], [[BE-NOTES/Python/OOP/Incapsulamento e Property|Property]], [[BE-NOTES/Python/OOP/Magic Methods|Magic Methods]].

## Definizione e istanze

```python
class Persona:
    """Rappresenta una persona."""

    # Attributo di classe (condiviso tra tutte le istanze)
    specie = "Homo sapiens"

    def __init__(self, nome: str, eta: int):
        """Costruttore — si chiama quando crei l'istanza."""
        self.nome = nome   # attributo di istanza
        self.eta = eta     # attributo di istanza

    def descrizione(self) -> str:
        """Metodo di istanza — primo parametro è self."""
        return f"{self.nome} ha {self.eta} anni"

    @classmethod
    def da_nascita(cls, nome: str) -> "Persona":
        """Metodo di classe — riceve cls invece di self."""
        return cls(nome, 0)

    @staticmethod
    def è_maggiorenne(eta: int) -> bool:
        """Metodo statico — non riceve né self né cls."""
        return eta >= 18


# Creare istanze
mario = Persona("Mario", 25)
print(mario.descrizione())    # "Mario ha 25 anni"
print(mario.specie)           # "Homo sapiens" (da classe)

# Metodo di classe
neonato = Persona.da_nascita("Anna")
```

## self e __init__

- `self`: riferimento all'istanza corrente (come `this` in Java/C#)
- `__init__`: costruttore, inizializza l'istanza. NON è il costruttore
  vero (quello è `__new__`, raramente sovrascritto)
- Puoi aggiungere attributi dinamicamente dopo `__init__` (ma non farlo)

## Visibilità

Python non ha `private` / `public` — usa convenzioni:

```python
class Conto:
    def __init__(self, saldo: float):
        self._saldo = saldo    # convention: "protetto" (uso interno)
        self.__pin = "1234"    # name mangling: _Conto__pin (evita conflitti)

    def mostra_saldo(self) -> float:
        return self._saldo
```

## Type hints per Self e classi

```python
from __future__ import annotations  # Python 3.7+

class Nodo:
    def collega(self, altro: "Nodo") -> None:  # forward reference
        self.prossimo = altro

# Python 3.11+: Self type
from typing import Self

class Costruttore:
    @classmethod
    def create(cls) -> Self:
        return cls()
```

## Best practice — spiegazioni

### Composizione > Ereditarietà: cosa significa

Questo principio (Favor composition over inheritance) dice: **costruisci oggetti assemblando componenti**, invece di creare gerarchie di classi.

Il problema con l'ereditarietà:
```python
# Ereditarietà — fragile, accoppiamento forte
class Automobile:
    def muovi(self): ...
class AutomobileVolante(Automobile):  # eredita muovi
    def vola(self): ...
class AutomobileVolanteAnfibia(AutomobileVolante):  # eredita muovi+vola
    def nuota(self): ...
# Dopo 3 livelli: "diamond problem", override imprevedibili
```

La soluzione con composizione:
```python
# Composizione — flessibile, disaccoppiato
class Motore:
    def muovi(self): ...
class Ali:
    def vola(self): ...
class Elica:
    def nuota(self): ...

class Veicolo:
    def __init__(self):
        self.motore = Motore()
        self.ali = Ali()
        self.elica = Elica()

v = Veicolo()
v.motore.muovi()  # usa componenti, non eredita
```
Benefici: testi ogni componente separatamente, cambi un componente senza rompere gli altri, nessun "diamond problem".

### Non usare `@dataclass` per repr/eq automatici se serve comportamento custom

## Security

- **`__pin` (name mangling) non è sicurezza** — è accessibile come `_Conto__pin`
- **Override di `__eq__` richiede `__hash__`** — altrimenti l'oggetto diventa non hashable
- **`__del__` non è garanzia di cleanup** — usa context manager ([[BE-NOTES/Python/Tecnologie/Context Manager|Context Manager]])
