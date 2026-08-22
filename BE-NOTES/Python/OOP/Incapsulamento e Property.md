---
topic: "Incapsulamento e Property — Python OOP"
tags: [python, oop, encapsulation, property]
nav_prev: "[[Classi e Oggetti.md]]"
nav_next: "[[Ereditariet�.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/functions.html#property](https://docs.python.org/3/library/functions.html#property)

Python non ha `private`/`protected` come Java/C#. L'incapsulamento è basato su **convenzioni** e su `@property` per getter/setter controllati.

`@property` è uno dei [[BE-NOTES/Python/OOP/Decoratori|decoratori]] built-in più usati. Trasforma un metodo in attributo "magico": si accede come attributo ma può avere logica di validazione, calcolo, caching.

Vedi anche: [[BE-NOTES/Python/OOP/Magic Methods|Magic Methods]] per `__getattr__`/`__setattr__`, [[BE-NOTES/Python/Funzionale/functools|cached_property]].

## Incapsulamento — convenzioni

Python non ha modificatori di accesso (`private`, `protected`). Usa convenzioni:

```python
class ContoBancario:
    def __init__(self, titolare: str, saldo: float = 0.0):
        self.titolare = titolare  # pubblico
        self._saldo = saldo       # "protetto" (uso interno, convenzione)
        self.__pin = "0000"       # "privato" (name mangling)
```

## @property

Permette di usare metodi come attributi, convalidando l'accesso:

```python
class Termometro:
    def __init__(self):
        self._celsius = 0

    @property
    def celsius(self) -> float:
        """Getter — si usa come attributo."""
        return self._celsius

    @celsius.setter
    def celsius(self, valore: float) -> None:
        """Setter — convalidazione prima di assegnare."""
        if valore < -273.15:
            raise ValueError("Sotto lo zero assoluto!")
        self._celsius = valore

    @property
    def fahrenheit(self) -> float:
        """Sola lettura — nessun setter."""
        return self._celsius * 9/5 + 32


t = Termometro()
t.celsius = 25          # usa il setter
print(t.celsius)        # 25 (usa il getter)
print(t.fahrenheit)     # 77.0
# t.fahrenheit = 80     # AttributeError: no setter
```

## Property vs attributo pubblico

| `@property` | Attributo semplice |
|---|---|
| Validazione input | Nessuna validazione |
| Calcolo derivato | Valore diretto |
| Backward compatibility | Cambiare pubblico rompe API |
| Accesso controllato | Accesso diretto |

## cached_property (Python 3.8+)

```python
from functools import cached_property

class Analisi:
    def __init__(self, dati: list[int]):
        self.dati = dati

    @cached_property
    def media(self) -> float:
        """Calcolato una volta, poi cache automatica."""
        return sum(self.dati) / len(self.dati)
```
