---
topic: "Database Connection — SQLAlchemy"
tags: [python, database, sqlalchemy, connection]
nav_next: "[[SQLAlchemy Engine e Core.md]]"
---
Riferimento ufficiale: [docs.sqlalchemy.org/en/20/core/engines.html](https://docs.sqlalchemy.org/en/20/core/engines.html)

L'`Engine` è il punto di ingresso di SQLAlchemy: gestisce connection pool, dialetto SQL, URL del database. Va creato **una volta** all'avvio dell'applicazione e riusato per tutta la vita.

La stringa di connessione segue il formato: `dialetto+driver://user:pass@host:port/dbname`

Vedi anche: [[BE-NOTES/Python/Data/SQLAlchemy Engine e Core|SQLAlchemy Core]], [[BE-NOTES/Python/Data/SQLAlchemy ORM|SQLAlchemy ORM]], [[BE-NOTES/Python/Data/Alembic|Alembic]] per migrazioni.

`create_engine` non apre subito una connessione: crea un **Engine** che gestisce un pool di connessioni e le apre al primo utilizzo. La stringa di connessione segue il formato `dialetto+driver://user:pass@host:port/dbname`. `+driver` è opzionale (se omesso, SQLAlchemy usa il driver predefinito per quel dialetto). Per SQLite, `///./mio.db` = file locale, `///:memory:` = database in RAM.

```python
from sqlalchemy import create_engine

# SQLite
engine = create_engine("sqlite:///./mio.db")
engine = create_engine("sqlite:///:memory:")  # in-memory

# PostgreSQL
engine = create_engine("postgresql://user:pass@localhost:5432/dbname")

# MySQL
engine = create_engine("mysql+pymysql://user:pass@localhost:3306/dbname")

# Oracle
engine = create_engine("oracle+cx_oracle://user:pass@localhost:1521/sid")
```

## Connection pool

Per applicazioni web è fondamentale usare un pool. `pool_size=5` mantiene 5 connessioni aperte sempre; `max_overflow=10` permette fino a 10 connessioni extra sotto carico (totale 15). `pool_pre_ping=True` esegue `SELECT 1` prima di usare una connessione dal pool — evita errori su connessioni stale. `pool_recycle=3600` forza la chiusura e riapertura dopo 1 ora (bypassa timeout firewall/DB).

```python
# Per applicazioni web (FastAPI, Django)
engine = create_engine(
    "postgresql://user:pass@localhost/dbname",
    pool_size=5,           # connessioni mantenute aperte
    max_overflow=10,       # extra oltre pool_size
    pool_pre_ping=True,    # verifica connessione prima di usarla
    pool_recycle=3600,     # ricicla dopo 1h
)
```

## Eseguire query raw

`text()` avvolge una stringa SQL in un oggetto eseguibile da SQLAlchemy. I parametri usano `:nome` (stile `:named`) — mai interpolare stringhe (SQL injection). `conn.execute()` restituisce un `CursorResult` iterabile. `row.id` accede alle colonne per attributo (oltre che per indice numerico o nome). `conn.commit()` è necessario per `INSERT`/`UPDATE`/`DELETE`; `SELECT` non richiede commit.

```python
from sqlalchemy import text

with engine.connect() as conn:
    # SELECT
    result = conn.execute(text("SELECT * FROM utenti"))
    for row in result:
        print(row.id, row.nome)

    # INSERT con parametri
    conn.execute(
        text("INSERT INTO utenti (nome, email) VALUES (:nome, :email)"),
        {"nome": "Mario", "email": "mario@test.it"}
    )
    conn.commit()  # necessario per scritture

    # Con context manager, commit automatico su successo
    # rollback automatico su eccezione
```

## Transazioni

`engine.begin()` è la forma raccomandata: apre una connessione, avvia una transazione, fa commit se il blocco termina senza eccezioni, rollback altrimenti — tutto automatico. `engine.connect()` + `conn.begin()` è la forma esplicita: utile se devi decidere a metà blocco se commitare o rollbackare in base a condizioni runtime.

```python
# Implicita (autocommit con context manager)
with engine.begin() as conn:
    conn.execute(text("INSERT INTO ..."))
    conn.execute(text("UPDATE ..."))
    # commit automatico alla fine del with
    # rollback se eccezione

# Esplicita
with engine.connect() as conn:
    trans = conn.begin()
    try:
        conn.execute(text("DELETE FROM ..."))
        trans.commit()
    except:
        trans.rollback()
        raise
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `OperationalError: no such table` | Tabelle non create | Esegui `Base.metadata.create_all(engine)` o migrazioni |
| `TimeoutError` su pool esaurito | Richieste > `pool_size + max_overflow` | Aumenta pool o ottimizza query |
| Connessione caduta dopo inattività | `pool_pre_ping=False` e firewall kill | Imposta `pool_pre_ping=True` e `pool_recycle=3600` |
| `IntegrityError` su INSERT | Violazione FK/unique | Controlla i dati o usa `INSERT ... ON CONFLICT` |
| Password con caratteri speciali nell'URL | URL non correttamente encoded | Usa `URL.create()` da `sqlalchemy.engine`
