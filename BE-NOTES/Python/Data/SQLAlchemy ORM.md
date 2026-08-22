---
topic: "SQLAlchemy ORM"
tags: [python, sqlalchemy, orm, database]
nav_prev: "[[SQLAlchemy Engine e Core.md]]"
nav_next: "[[Alembic.md]]"
---
Riferimento ufficiale: [docs.sqlalchemy.org/en/20/orm](https://docs.sqlalchemy.org/en/20/orm/)

SQLAlchemy ORM è un **Data Mapper** (a differenza dell'active record di Django). Separa il modello Python dalla riga DB — devi esplicitamente `session.add()` e `session.commit()`.

### SQLAlchemy ORM vs Django ORM

| SQLAlchemy | Django ORM |
|---|---|
| Data Mapper | Active Record |
| Session esplicita | Query implicita |
| Più complesso | Semplice |
| Async nativo (2.0) | Async via 3rd party |
| Più performante | Meno configurabile |

Vedi anche: [[BE-NOTES/Python/Data/Database Connection|Connection]], [[BE-NOTES/Python/Data/Alembic|Alembic]], [[BE-NOTES/Python/FastAPI/Database e SQLAlchemy|FastAPI + SQLAlchemy]].

`DeclarativeBase` (SQLAlchemy 2.0) è la classe base per tutti i modelli ORM. Eredita da essa e ogni sua sottoclasse diventa una tabella. `Mapped[str]` dichiara il tipo Python della colonna — SQLAlchemy lo usa per inferire il tipo SQL se non specificato da `mapped_column`. `Session` è l'oggetto che traccia le modifiche e le persiste (Unit of Work).

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

class Base(DeclarativeBase):
    pass

engine = create_engine("sqlite:///./test.db")
```

`__tablename__` definisce il nome della tabella nel DB. `Mapped[int]` con `mapped_column(primary_key=True)` dichiara una colonna intera con PK. `relationship(back_populates="utente")` crea un collegamento Python tra le classi — non crea colonne ma permette `utente.ordini` e `ordine.utente`. `Optional[int]` permette NULL in DB. `__repr__` definisce la rappresentazione stringa dell'oggetto.

## Modelli ORM (SQLAlchemy 2.0 style)

```python
from typing import Optional
from sqlalchemy import String, ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, relationship

class Utente(Base):
    __tablename__ = "utenti"

    id: Mapped[int] = mapped_column(primary_key=True)
    nome: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(unique=True)
    eta: Mapped[Optional[int]]

    # Relazione
    ordini: Mapped[list["Ordine"]] = relationship(back_populates="utente")

    def __repr__(self) -> str:
        return f"Utente({self.nome})"

class Ordine(Base):
    __tablename__ = "ordini"

    id: Mapped[int] = mapped_column(primary_key=True)
    utente_id: Mapped[int] = mapped_column(ForeignKey("utenti.id"))
    totale: Mapped[float]
    descrizione: Mapped[Optional[str]]

    utente: Mapped["Utente"] = relationship(back_populates="ordini")

Base.metadata.create_all(engine)
```

## CRUD ORM

`Session(engine)` apre una sessione — tiene traccia di tutti gli oggetti modificati. `session.add()` inserisce l'oggetto nella sessione (INSERT differito). `session.commit()` esegue tutte le operazioni in sospeso in una transazione — dopo il commit, `mario.id` viene popolato con l'ID generato dal DB. `session.get(Utente, 1)` carica per PK (prima cerca nella sessione, poi eventualmente nel DB). `session.query()` è la vecchia API (funziona ma la 2.0 preferisce `select()`). `.first()` restituisce il primo risultato o `None`.

```python
from sqlalchemy.orm import Session

# CREATE
with Session(engine) as session:
    mario = Utente(nome="Mario", email="mario@test.it")
    session.add(mario)
    session.commit()  # qui mario.id viene popolato

# READ
with Session(engine) as session:
    utente = session.get(Utente, 1)  # by PK
    utenti = session.query(Utente).filter(Utente.eta > 18).all()
    utente = session.query(Utente).where(Utente.nome == "Mario").first()

# UPDATE
with Session(engine) as session:
    mario = session.get(Utente, 1)
    mario.eta = 26
    session.commit()

# DELETE
with Session(engine) as session:
    mario = session.get(Utente, 1)
    session.delete(mario)
    session.commit()
```

## Relazioni eager vs lazy

Per default le relazioni sono **lazy**: `utente.ordini` esegue una query al primo accesso. Problema N+1: se carichi 100 utenti e accedi a `ordini` per ciascuno, fai 1 + 100 query. `joinedload()` risolve con LEFT JOIN in una sola query (ma può duplicare righe con ManyToOne — usa `distinct()`). `selectinload()` fa una seconda query con `WHERE id IN (...)` — spesso più efficiente di JOIN per relazioni ricche.

```python
# Lazy (default) — carica al primo accesso
utente = session.get(Utente, 1)
print(utente.ordini)  # esegue SELECT qui

# Eager — carica tutto subito (evita N+1)
from sqlalchemy.orm import joinedload

utente = session.query(Utente).options(
    joinedload(Utente.ordini)
).filter(Utente.id == 1).one()
print(utente.ordini)  # già caricato

# selectinload (spesso meglio di joinedload per ManyToOne)
from sqlalchemy.orm import selectinload
utente = session.query(Utente).options(
    selectinload(Utente.ordini)
).first()
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `DetachedInstanceError` | Accesso a lazy load dopo chiusura sessione | Carica tutto dentro il `with Session()` o usa `joinedload()` |
| `MultipleResultsFound` | `.one()` restituisce più righe | Usa `.first()` o `.scalar()` o aggiungi filtri |
| `FlushError` su oggetto già persistito | Stessa PK già nella sessione | Usa `session.merge()` invece di `session.add()` |
| N+1 queries | Lazy load su lista di oggetti | Aggiungi `selectinload()` o `joinedload()` |
| `ObjectNotPersisted` | Tentativo di accedere a ID di oggetto non ancora flushato | Chiama `session.flush()` per popolare l'ID
