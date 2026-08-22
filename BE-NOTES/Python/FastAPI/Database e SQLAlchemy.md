---
topic: "Database e SQLAlchemy — FastAPI"
tags: [python, fastapi, sqlalchemy, database, orm]
nav_prev: "[[Router e Middleware.md]]"
nav_next: "[[Auth e Sicurezza.md]]"
---
Riferimento ufficiale: [fastapi.tiangolo.com/tutorial/sql-databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)

Integrazione FastAPI + SQLAlchemy 2.0. Pattern standard: **session-per-request** tramite `Depends(get_db)`. Ogni request apre una sessione, la usa, e la chiude automaticamente.

Vedi anche: [[BE-NOTES/Python/Data/SQLAlchemy ORM|SQLAlchemy ORM]] (approfondimento modelli), [[BE-NOTES/Python/Data/Alembic|Alembic]] (migrazioni), [[BE-NOTES/Python/FastAPI/Dependency Injection|DI]] per `get_db`.

`create_engine()` crea il motore di connessione al database — un pool di connessioni che SQLAlchemy gestisce automaticamente. `sessionmaker()` genera una fabbrica di sessioni: ogni chiamata `SessionLocal()` apre una nuova sessione (una connessione dal pool). `declarative_base()` restituisce una classe base per i modelli ORM. `connect_args={"check_same_thread": False}` è necessario solo per SQLite, che di default permette accessi solo dallo stesso thread.

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

DATABASE_URL = "sqlite:///./test.db"
# PostgreSQL: "postgresql://user:pass@localhost/dbname"
# MySQL:      "mysql+pymysql://user:pass@localhost/dbname"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}  # solo SQLite
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

## Modello DB

```python
from sqlalchemy import Column, Integer, String

class PersonaDB(Base):
    __tablename__ = "persone"

    id = Column(Integer, primary_key=True, index=True)
    nome = Column(String, index=True)
    eta = Column(Integer)
    città = Column(String, default="Roma")

Base.metadata.create_all(bind=engine)  # crea tabelle
```

## Schema Pydantic per API

```python
from pydantic import BaseModel

class PersonaSchema(BaseModel):
    nome: str
    eta: int
    città: str = "Roma"

    class Config:
        from_attributes = True  # allow ORM mode
```

## Dipendenza DB

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

## CRUD completo

Il flusso standard di ogni operazione CRUD: (1) ricevi i dati validati da Pydantic, (2) converti in modello ORM, (3) esegui l'operazione sul DB, (4) committa, (5) restituisci il risultato. `response_model=PersonaSchema` fa sì che FastAPI serializzi l'istanza ORM usando lo schema Pydantic, filtrando i campi non inclusi. `db.refresh()` ricarica l'oggetto dal DB (utile per avere ID autogenerati o default del DB).

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session

app = FastAPI()

# CREATE
@app.post("/persone", response_model=PersonaSchema)
def crea_persona(persona: PersonaSchema, db: Session = Depends(get_db)):
    db_persona = PersonaDB(**persona.model_dump())
    db.add(db_persona)
    db.commit()
    db.refresh(db_persona)
    return db_persona

# READ (one)
@app.get("/persone/{id}", response_model=PersonaSchema)
def get_persona(id: int, db: Session = Depends(get_db)):
    persona = db.query(PersonaDB).filter(PersonaDB.id == id).first()
    if not persona:
        raise HTTPException(status_code=404, detail="Non trovato")
    return persona

# READ (list)
@app.get("/persone", response_model=list[PersonaSchema])
def lista_persone(db: Session = Depends(get_db)):
    return db.query(PersonaDB).all()

# UPDATE
@app.put("/persone/{id}", response_model=PersonaSchema)
def aggiorna_persona(id: int, persona: PersonaSchema, db: Session = Depends(get_db)):
    db_persona = db.query(PersonaDB).filter(PersonaDB.id == id).first()
    if not db_persona:
        raise HTTPException(status_code=404, detail="Non trovato")
    for key, value in persona.model_dump().items():
        setattr(db_persona, key, value)
    db.commit()
    db.refresh(db_persona)
    return db_persona

# DELETE
@app.delete("/persone/{id}")
def elimina_persona(id: int, db: Session = Depends(get_db)):
    db_persona = db.query(PersonaDB).filter(PersonaDB.id == id).first()
    if not db_persona:
        raise HTTPException(status_code=404, detail="Non trovato")
    db.delete(db_persona)
    db.commit()
    return {"messaggio": "Eliminato"}
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `sqlalchemy.exc.IntegrityError` | Violazione vincolo DB (es. unique, FK) | Controlla dati prima di committare |
| `DetachedInstanceError` dopo refresh | Oggetto non più legato alla sessione | Mantieni la sessione aperta durante la serializzazione |
| `MissingGreenlet` con async | Usato `async def` con SQLAlchemy sync | Usa `def` invece di `async def` o installa `greenlet` |
| `MultipleResultsFound` con `.first()` | `.one()` lancia eccezione se ci sono più risultati | Usa `.first()` che restituisce `None` se non trovato |
| Lentezza su liste grandi | `.all()` carica TUTTE le righe in memoria | Usa `.offset().limit()` per paginare |

## Best practice

- **Session-per-request**: mai condividere una sessione tra request diverse
- **`response_model` sempre**: filtra i campi sensibili (es. password hash) dall'output API
- **Indici**: aggiungi `index=True` sui campi usati in `filter()` o `JOIN` 
- **Paginazione**: per liste usa sempre `skip`/`limit` o cursor-based pagination
- **Enum**: usa `Enum` di Python + `sa.Enum` nei modelli, non stringhe
