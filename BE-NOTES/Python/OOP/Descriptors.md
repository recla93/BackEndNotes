---
topic: "Descriptors — Python OOP"
tags: [python, oop, descriptors, property, attribute-access]
---
Riferimento ufficiale: [docs.python.org/3/howto/descriptor.html](https://docs.python.org/3/howto/descriptor.html)

Un descrittore e un oggetto che definisce `__get__`, `__set__` o `__delete__`, controllando come un attributo viene accessibile su un'istanza. E il meccanismo sottostante a `@property`, `@staticmethod`, `@classmethod` e `__slots__`.

Quando accedi a `obj.attr`, Python cerca l'attributo prima nella classe e nei suoi descrittori, poi nell'istanza. Se la classe ha un descrittore con `__get__`, viene chiamato quello -- non l'attributo dell'istanza.

Vedi anche:
[[BE-NOTES/Python/OOP/Incapsulamento e Property|Incapsulamento e Property]],
[[BE-NOTES/Python/OOP/Decoratori|Decoratori]],
[[BE-NOTES/Python/OOP/Magic Methods|Magic Methods]].

## Descrittore base

```python
class PositiveNumber:
    """Descrittore che forza valori positivi."""

    def __set_name__(self, owner: type, name: str):
        """Chiamato quando il descrittore viene assegnato in una classe."""
        self._name = name

    def __get__(self, obj: object | None, objtype: type | None = None):
        if obj is None:
            return self
        return obj.__dict__.get(self._name, 0)

    def __set__(self, obj: object, value: int | float):
        if value < 0:
            raise ValueError(f"{self._name} deve essere positivo")
        obj.__dict__[self._name] = value


class ContoBancario:
    saldo = PositiveNumber()  # Descrittore a livello di classe

    def __init__(self, saldo_iniziale: float):
        self.saldo = saldo_iniziale  # Chiama PositiveNumber.__set__


conto = ContoBancario(100.0)
print(conto.saldo)     # 100.0 (chiama __get__)
conto.saldo = -50      # ValueError: saldo deve essere positivo
```

`__set_name__` (Python 3.6+) riceve automaticamente il nome dell'attributo nella classe. I descrittori salvano i valori in `obj.__dict__` per evitare ricorsione infinita.

## Tipi di descrittori

```python
# Descrittore con solo __get__ (non-data descriptor)
class ReadOnly:
    def __get__(self, obj, objtype=None):
        return 42

# Descrittore con __set__ (data descriptor)
class Validated:
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get('_val', 0)

    def __set__(self, obj, value):
        obj.__dict__['_val'] = value
```

I **data descriptor** (con `__set__` o `__delete__`) hanno priorita su `obj.__dict__`. I **non-data descriptor** (solo `__get__`) hanno priorita minore degli attributi d'istanza.

## property come descrittore

```python
class Persona:
    def __init__(self, nome: str):
        self._nome = nome

    @property
    def nome(self) -> str:
        """Descrittore built-in."""
        return self._nome

    @nome.setter
    def nome(self, valore: str):
        if not valore.strip():
            raise ValueError("Nome non vuoto")
        self._nome = valore

    @nome.deleter
    def nome(self):
        raise AttributeError("Non puoi eliminare il nome")
```

`@property` e syntactic sugar per un descrittore. Crea un descrittore di dati con `__get__`, `__set__` e `__delete__` opzionali.

## Validazione con descrittori

```python
class Range:
    """Descrittore che valuta un valore in un intervallo."""

    def __init__(self, min_val: float, max_val: float):
        self.min_val = min_val
        self.max_val = max_val

    def __set_name__(self, owner, name: str):
        self._name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self._name, self.min_val)

    def __set__(self, obj, value):
        if not (self.min_val <= value <= self.max_val):
            raise ValueError(
                f"{self._name} deve essere tra {self.min_val} e {self.max_val}"
            )
        setattr(obj, self._name, value)


class Prodotto:
    prezzo = Range(0.01, 10000.0)
    sconto = Range(0.0, 100.0)

    def __init__(self, prezzo: float, sconto: float):
        self.prezzo = prezzo
        self.sconto = sconto
```

I descrittori sono riutilizzabili: uno stesso `Range` puo validare sia `prezzo` che `sconto`. Pattern utile per modelli di dati con validazione.

## Lazy property con descrittore

```python
class LazyProperty:
    """Descrittore che calcola il valore una volta sola."""

    def __init__(self, func):
        self.func = func
        self._name = None

    def __set_name__(self, owner, name):
        self._name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        # Calcola e cache
        value = self.func(obj)
        setattr(obj, self._name, value)
        # Sostituisce se stesso con il valore calcolato
        obj.__dict__[self._name] = value
        return value


class Report:
    def __init__(self, data: list[int]):
        self.data = data

    @LazyProperty  # type: ignore
    def average(self) -> float:
        """Calcolato una volta, poi cachato."""
        print("Calcolo media...")
        return sum(self.data) / len(self.data)


r = Report([10, 20, 30])
print(r.average)  # Calcolo media... 20.0
print(r.average)  # 20.0 (dal cache, non ricalcola)
```

Dopo il primo accesso, il descrittore salva il valore in `obj.__dict__`. Al secondo accesso, `obj.__dict__` ha priorita sul non-data descriptor (se non implementa `__set__`).

## `__set_name__` e introspezione

```python
class TrackDescriptors:
    def __set_name__(self, owner, name):
        print(f"Creato {name} in {owner.__name__}")

class Esempio:
    x = TrackDescriptors()  # Stampa: Creato x in Esempio
    y = TrackDescriptors()  # Stampa: Creato y in Esempio
```

`__set_name__` e chiamato automaticamente quando la classe viene creata, subito dopo `type.__new__`. Sostituisce la vecchia pratica di passare il nome al costruttore.

## Errori comuni

- **Ricorsione infinita**: se `__get__` accede a `self.attr` invece di `self.__dict__['attr']`.
- **Dimenticare `__set_name__`**: nei vecchi descrittori dovevi specificare il nome manualmente.
- **Non-data descriptor per validazione**: senza `__set__`, l'assegnazione diretta su `obj.attr` bypassa il descrittore.
- **Condividere stato tra istanze**: se il descrittore ha attributi mutabili modificati in `__set__`, attento alla visibilita tra istanze.
- **Troppa astrazione**: per validazione semplice, una `@property` basta. I descrittori servono per logiche riutilizzabili.

## Best Practices & Conventions

- Usa **`@property`** per validazione semplice di un singolo attributo.
- Usa **descrittori** per logiche di validazione **riutilizzabili** su piu classi/attributi.
- Implementa sempre `__set_name__` (Python 3.6+) invece di passare il nome nel costruttore.
- Per lazy initialization, valuta `functools.cached_property` (Python 3.9+) prima di scrivere un descrittore custom.
- I descrittori sono ortogonali alle metaclassi: i primi lavorano sugli attributi, le seconde sulla creazione della classe.
- Documenta il comportamento del descrittore: `__get__`, `__set__`, `__delete__` e side effects.
