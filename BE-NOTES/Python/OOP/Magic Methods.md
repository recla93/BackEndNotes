---
topic: "Magic Methods — Python OOP"
tags: [python, oop, dunder, magic-methods, operators]
nav_prev: "[[Ereditariet�.md]]"
nav_next: "[[Decoratori.md]]"
---
Riferimento ufficiale: [docs.python.org/3/reference/datamodel.html#special-method-names](https://docs.python.org/3/reference/datamodel.html#special-method-names)

## Perché "magic"?

I metodi con doppio underscore (`__metodo__`) consentono di ridefinire
come gli oggetti interagiscono con operatori, built-in e sintassi
di Python. Non vanno chiamati direttamente — Python li chiama da solo.

A differenza di Java (interface per OperatorOverload), Python permette di
ridefinire praticamente ogni aspetto: aritmetica, indicizzazione, iterazione,
chiamate, context manager, conversione a stringa.

Vedi anche: [[BE-NOTES/Python/OOP/Classi e Oggetti|Classi]], [[BE-NOTES/Python/OOP/Incapsulamento e Property|Property]], [[BE-NOTES/Python/Tecnologie/Context Manager|Context Manager]] per `__enter__`/`__exit__`.

## Creazione e rappresentazione

```python
class Punto:
    def __init__(self, x: float, y: float):
        """Inizializzazione (costruttore)."""
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        """Rappresentazione per debug."""
        return f"Punto({self.x}, {self.y})"

    def __str__(self) -> str:
        """Rappresentazione per utente (print)."""
        return f"({self.x}, {self.y})"

    def __eq__(self, altro: object) -> bool:
        """== permette confronto."""
        if not isinstance(altro, Punto):
            return NotImplemented
        return self.x == altro.x and self.y == altro.y

    def __hash__(self) -> int:
        """Rende l'oggetto usabile come chiave dict / set."""
        return hash((self.x, self.y))

    def __bool__(self) -> bool:
        """Falsy/truthy: bool(punto)."""
        return self.x != 0 or self.y != 0
```

## Operatori aritmetici

```python
class Vettore:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __add__(self, altro: "Vettore") -> "Vettore":
        return Vettore(self.x + altro.x, self.y + altro.y)

    def __sub__(self, altro: "Vettore") -> "Vettore":
        return Vettore(self.x - altro.x, self.y - altro.y)

    def __mul__(self, scala: float) -> "Vettore":
        return Vettore(self.x * scala, self.y * scala)

    def __repr__(self) -> str:
        return f"Vettore({self.x}, {self.y})"

v1 = Vettore(1, 2)
v2 = Vettore(3, 4)
print(v1 + v2)  # Vettore(4, 6)  — chiama __add__
print(v1 * 3)   # Vettore(3, 6)  — chiama __mul__
```

## Container

```python
class Mappa:
    def __init__(self):
        self._dati: dict[str, int] = {}

    def __getitem__(self, key: str) -> int:
        """Supporta mappa["chiave"]."""
        return self._dati[key]

    def __setitem__(self, key: str, value: int):
        """Supporta mappa["chiave"] = valore."""
        self._dati[key] = value

    def __contains__(self, key: str) -> bool:
        """Supporta "chiave" in mappa."""
        return key in self._dati

    def __len__(self) -> int:
        """Supporta len(mappa)."""
        return len(self._dati)

    def __iter__(self):
        """Supporta for key in mappa."""
        return iter(self._dati)
```

## Callable

```python
class Moltiplicatore:
    def __init__(self, fattore: float):
        self.fattore = fattore

    def __call__(self, valore: float) -> float:
        """Rende l'istanza chiamabile come funzione."""
        return valore * self.fattore

per_2 = Moltiplicatore(2)
per_2(5)  # 10 (__call__)
```

## Tabella riassuntiva

| Metodo | Attivato da |
|---|---|
| `__init__` | `Classe()` |
| `__repr__` | `repr(obj)` |
| `__str__` | `print(obj)`, `str(obj)` |
| `__eq__` | `obj == altro` |
| `__hash__` | `hash(obj)` |
| `__bool__` | `bool(obj)`, `if obj:` |
| `__add__`, `__sub__`, `__mul__` | `+`, `-`, `*` |
| `__getitem__`, `__setitem__` | `obj[k]`, `obj[k] = v` |
| `__contains__` | `x in obj` |
| `__len__` | `len(obj)` |
| `__iter__` | `for x in obj` |
| `__call__` | `obj(args)` |
| `__enter__`, `__exit__` | `with obj:` |
