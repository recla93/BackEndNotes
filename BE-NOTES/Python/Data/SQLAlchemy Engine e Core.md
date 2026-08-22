---
topic: "SQLAlchemy Engine e Core"
tags: [python, sqlalchemy, database, core]
nav_prev: "[[Database Connection.md]]"
nav_next: "[[SQLAlchemy ORM.md]]"
---
Riferimento ufficiale: [docs.sqlalchemy.org/en/20/core](https://docs.sqlalchemy.org/en/20/core/)

SQLAlchemy Core è l'astrazione SQL **low-level**: costruisci query con oggetti Python ma senza ORM. Più verboso dell'ORM ma più performante e con controllo granulare su SQL generato.

Usa Core quando: operazioni bulk, CTE complesse, query dinamiche, necessità di JOIN non standard. Usa ORM ([[BE-NOTES/Python/Data/SQLAlchemy ORM|vedi]]) per CRUD standard e business logic.

Vedi anche: [[BE-NOTES/Python/Data/Database Connection|Connection]], [[BE-NOTES/Python/Data/Alembic|Alembic]].

`Table` definisce una tabella a livello di schema — non è un modello ORM ma un oggetto che rappresenta la struttura della tabella. `MetaData` è un contenitore di schema che lega tutte le `Table` insieme. `metadata.create_all(engine)` genera le istruzioni `CREATE TABLE` nel DB — confronta `metadata` con le tabelle esistenti e crea solo quelle mancanti.

```python
from sqlalchemy import (
    Table, Column, Integer, String, Float,
    MetaData, ForeignKey, create_engine, select
)

metadata = MetaData()

utenti = Table(
    "utenti", metadata,
    Column("id", Integer, primary_key=True),
    Column("nome", String(100), nullable=False),
    Column("email", String(255), unique=True),
)

ordini = Table(
    "ordini", metadata,
    Column("id", Integer, primary_key=True),
    Column("utente_id", Integer, ForeignKey("utenti.id")),
    Column("totale", Float),
)

engine = create_engine("sqlite:///./test.db")
metadata.create_all(engine)  # crea tabelle
```

## CRUD con Core

Le funzioni `insert()`, `select()`, `update()`, `delete()` costruiscono oggetti query che vengono compilati in SQL al momento dell'esecuzione. `utenti.c.nome` accede alla colonna `nome` della tabella `utenti` (`.c` = columns). `engine.begin()` gestisce automaticamente commit/rollback per scritture. `for row in result` itera sulle righe — ogni riga è un oggetto `Row` con accesso per attributo (`row.nome`) o indice (`row[0]`).

```python
from sqlalchemy import insert, update, delete, select

# CREATE
with engine.begin() as conn:
    conn.execute(
        insert(utenti).values(nome="Mario", email="mario@test.it")
    )

# READ
with engine.connect() as conn:
    result = conn.execute(
        select(utenti).where(utenti.c.nome == "Mario")
    )
    for row in result:
        print(row.id, row.nome)

# UPDATE
with engine.begin() as conn:
    conn.execute(
        update(utenti)
        .where(utenti.c.id == 1)
        .values(nome="Luigi")
    )

# DELETE
with engine.begin() as conn:
    conn.execute(
        delete(utenti).where(utenti.c.id == 1)
    )
```

## Join e query complesse

`utenti.join(ordini)` crea automaticamente un JOIN su `utenti.id = ordini.utente_id` basandosi sulla `ForeignKey`. `.select_from()` specifica esplicitamente la clausola FROM (necessaria per JOIN multipli). `func.sum()` avvolge una funzione SQL — `func` contiene tutte le funzioni standard (COUNT, AVG, MAX, COALESCE, ecc.). `.label()` assegna un alias alla colonna nel risultato.

```python
from sqlalchemy import join, func

# Join
query = (
    select(utenti.c.nome, ordini.c.totale)
    .select_from(utenti.join(ordini))
)

# Aggregazioni
query = (
    select(utenti.c.nome, func.sum(ordini.c.totale).label("speso_totale"))
    .select_from(utenti.join(ordini))
    .group_by(utenti.c.id)
)
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `NoSuchTableError` | Tabella referenziata ma non in `metadata` | Verifica che la `Table` sia associata al `MetaData` |
| `InvalidRequestError` su `.c.` | Accesso a colonna inesistente | Controlla i nomi con `tabella.columns.keys()` |
| `CompileError` su func | Funzione SQL non supportata dal dialetto | Usa `func` solo per funzioni standard SQL |
| `ResourceClosedError` | Accesso a result dopo chiusura connessione | Consuma il result dentro il `with` block |
| `FlushError` su INSERT bulk | Violazione unique constraint | Usa `insert(...).prefix_with("OR IGNORE")` |

## Core vs ORM

| Core | ORM |
|---|---|
| Query raw SQL-like | Query su oggetti Python |
| Più performante | Più produttivo |
| Controllo totale | Meno boilerplate |
| Per operazioni bulk | Per logica di business |
| Transazioni manuali | Unit of Work automatico |
