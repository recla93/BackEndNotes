---
topic: "Protocols e Structural Subtyping — Python"
tags: [python, oop, protocols, duck-typing, structural-typing, typing]
---
Riferimento ufficiale: [docs.python.org/3/library/typing.html#typing.Protocol](https://docs.python.org/3/library/typing.html#typing.Protocol)

Un `Protocol` (PEP 544, Python 3.8+) definisce un'interfaccia implicita basata su **struttura** invece che su ereditarieta. Se un oggetto ha i metodi e attributi dichiarati dal Protocol, e compatibile con quel Protocol - nessuna dichiarazione `extends` necessaria.

E l'evoluzione del duck typing ("se cammina come un'anatra e starnazza come un'anatra, allora e un'anatra") in un sistema di tipi statico controllabile da mypy/pyright. A differenza di `ABC` (Abstract Base Class) che richiede ereditarieta esplicita, un Protocol matcha per struttura.

Vedi anche:
[[BE-NOTES/Python/OOP/Classi e Oggetti|Classi e Oggetti]],
[[BE-NOTES/Python/OOP/Ereditarieta|Ereditarieta]],
[[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]].

## Protocol base

```python
from typing import Protocol

class Volabile(Protocol):
    def vola(self) -> str:
        ...

class Airone:
    def vola(self) -> str:
        return "L'airone vola alto"

class Aereo:
    def vola(self) -> str:
        return "L'aereo decolla"

    def atterra(self) -> str:
        return "Atterraggio"


def esegui_volo(oggetto: Volabile) -> None:
    print(oggetto.vola())


esegui_volo(Airone())  # OK: Airone ha vola()
esegui_volo(Aereo())   # OK: Aereo ha vola()

class Gatto:
    pass

# esegui_volo(Gatto())  # mypy: TypeError! Gatto non ha vola()
```

`Volabile` e un Protocol: dichiara cosa serve (`vola()`). `Airone` e `Aereo` sono compatibili senza ereditare da `Volabile`. mypy controlla la presenza dei metodi, non la gerarchia.

## Protocol con attributi

```python
from typing import Protocol

class Nominabile(Protocol):
    nome: str

    def saluta(self) -> str:
        ...

class Persona:
    def __init__(self, nome: str):
        self.nome = nome

    def saluta(self) -> str:
        return f"Ciao, sono {self.nome}"

class Robot:
    def __init__(self, seriale: str):
        self.nome = seriale

    def saluta(self) -> str:
        return f"Unita {self.nome} operativa"


def presentati(entita: Nominabile) -> None:
    print(entita.saluta())

presentati(Persona("Alice"))   # OK
presentati(Robot("R2D2"))      # OK
```

I Protocol possono dichiarare attributi (con type hint) oltre ai metodi. L'oggetto deve avere sia l'attributo che i metodi dichiarati.

## Protocol vs ABC

```python
from abc import ABC, abstractmethod
from typing import Protocol

# Approach ABC: ereditarieta esplicita
class AnimaleABC(ABC):
    @abstractmethod
    def suono(self) -> str: ...

class CaneABC(AnimaleABC):
    def suono(self) -> str:
        return "Bau"

# Approach Protocol: duck typing strutturale
class AnimaleProtocol(Protocol):
    def suono(self) -> str: ...

class CaneStruct:
    def suono(self) -> str:
        return "Bau"

class GattoStruct:
    def suono(self) -> str:
        return "Miao"
```

| Aspetto | ABC | Protocol |
|---------|-----|----------|
| Ereditarieta | Obbligatoria | Nessuna |
| Controllo runtime | `isinstance()` | No (solo type checker) |
| Flessibilita | Bassa | Alta |
| Intrusivita | Alta (modifica gerarchia) | Bassa |
| Scopo | Framework/chiusi | Librerie/esterni |

## Protocol compositi e ereditarieta

```python
from typing import Protocol

class Leggibile(Protocol):
    def leggi(self) -> str: ...

class Salvabile(Protocol):
    def salva(self, path: str) -> None: ...

class Documento(Leggibile, Salvabile, Protocol):
    """Protocol composito: deve avere sia leggi che salva."""
    ...


class FileTesto:
    def leggi(self) -> str:
        return "contenuto"

    def salva(self, path: str) -> None:
        print(f"Salvato in {path}")

def elabora(doc: Documento) -> None:
    contenuto = doc.leggi()
    print(f"Elaborato: {contenuto}")
```

I Protocol si combinano con ereditarieta multipla per creare Protocol compositi. `Documento` eredita da `Leggibile` e `Salvabile` ed e anch'esso un Protocol.

## @runtime_checkable

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class ConVolo(Protocol):
    def vola(self) -> str: ...

class Airone:
    def vola(self) -> str:
        return "Vola"

class Sasso:
    pass

# Solo con @runtime_checkable
print(isinstance(Airone(), ConVolo))  # True
print(isinstance(Sasso(), ConVolo))   # False
```

Per default `isinstance()` non funziona con Protocol. `@runtime_checkable` lo abilita, ma controlla solo la presenza dei metodi tramite `hasattr()`, non i type hint.

## Protocol generico

```python
from typing import Protocol, TypeVar

T = TypeVar('T')

class Confrontabile(Protocol[T]):
    def __lt__(self, other: T) -> bool: ...

class Persona:
    def __init__(self, nome: str, eta: int):
        self.nome = nome
        self.eta = eta

    def __lt__(self, other: "Persona") -> bool:
        return self.eta < other.eta


def massimo(seq: list[Confrontabile]) -> Confrontabile:
    """Trova il massimo in una sequenza confrontabile."""
    return max(seq)

persone = [Persona("A", 30), Persona("B", 25)]
m = massimo(persone)  # OK: Persona implementa __lt__
```

I Protocol generici permettono di definire vincoli di tipo flessibili, come `Confrontabile[T]` che richiede `__lt__`. Utile per algoritmi generici (ordinamento, ricerca).

## Protocol per dependency injection

```python
from typing import Protocol

class Database(Protocol):
    def query(self, sql: str) -> list[dict]: ...

class DatabasePostgres:
    def query(self, sql: str) -> list[dict]:
        # Connessione a PostgreSQL
        return [{"id": 1, "nome": "Alice"}]

class DatabaseMock:
    def query(self, sql: str) -> list[dict]:
        return [{"id": 1, "nome": "Mock"}]


class ServizioUtente:
    def __init__(self, db: Database):
        self._db = db

    def get_user(self, user_id: int) -> dict | None:
        result = self._db.query(f"SELECT * FROM utenti WHERE id = {user_id}")
        return result[0] if result else None


# Produzione
servizio = ServizioUtente(DatabasePostgres())

# Test (senza mock framework)
servizio_test = ServizioUtente(DatabaseMock())
```

I Protocol sono eccellenti per dependency injection: definisci l'interfaccia con un Protocol, poi fornisci implementazioni diverse per produzione e test. Nessuna dipendenza da framework DI.

## Errori comuni

- **Non usare `...` (Ellipsis) nei metodi**: il corpo del metodo in un Protocol deve essere `...` (o `pass`). Il corpo non viene mai eseguito.
- **Dimenticare che e solo per type checker**: a runtime, un Protocol non fornisce nessuna garanzia. Solo mypy/pyright lo controllano.
- **Troppi metodi in un Protocol**: un Protocol dovrebbe essere piccolo e focalizzato (Interface Segregation).
- **Confondere Protocol con ABC**: ABC da errori a runtime, Protocol solo al type checker. Usa ABC per codice che deve fallire presto.
- **`@runtime_checkable` lento**: `isinstance()` con Protocol controlla ogni attributo, non solo l'ereditarieta.

## Best Practices & Conventions

- Usa **Protocol** per definire interfacce in librerie e API pubbliche dove non vuoi imporre ereditarieta.
- Preferisci **ABC** quando l'ereditarieta e esplicita e vuoi controlli a runtime.
- Mantieni i Protocol piccoli: meglio 3 Protocol specializzati che 1 grande.
- Usa Protocol per **dependency injection** (soprattutto in testing) per evitare mock complessi.
- Combina Protocol con **TypeVar** per tipi generici flessibili.
- Non abusare di `@runtime_checkable`: rallenta e da falsi positivi (controlla solo `hasattr`, non i tipi).
