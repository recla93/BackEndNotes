---
topic: "Dependency Injection — FastAPI"
tags: [python, fastapi, di, dependencies]
nav_prev: "[[Pydantic e Validazione.md]]"
nav_next: "[[Router e Middleware.md]]"
---
Riferimento ufficiale: [fastapi.tiangolo.com/tutorial/dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)

FastAPI ha un sistema DI built-in potente. Ogni parametro di tipo `Depends()` viene risolto automaticamente. Supporta:
- **Gerarchia di dipendenze** (A dipende da B dipende da C)
- **Context manager via `yield`** (setup/cleanup automatico)
- **Override** per testing (sostituisci dipendenze in test)
- **Classi come dipendenze** (FastAPI istanzia automaticamente)

Vedi anche: [[BE-NOTES/Python/FastAPI/Database e SQLAlchemy|DB]] per `Depends(get_db)`, [[BE-NOTES/Python/FastAPI/Testing e Deploy|Testing]] per override, [[BE-NOTES/Python/Tecnologie/Context Manager|Context Manager]] per pattern yield.

`Depends()` è un marcatore per FastAPI: quando la route viene chiamata, FastAPI esegue la funzione passata a `Depends()`, ne cattura il valore restituito, e lo inietta come argomento dell'endpoint. In questo esempio, prima di eseguire `protetto()`, FastAPI chiama `get_token()`, che estrae l'header `x-token` e lo valida. Se il token è sbagliato, solleva `HTTPException` (403) — la route non viene mai eseguita. Se è valido, `token: str = Depends(get_token)` riceve il token come stringa.

```python
from fastapi import FastAPI, Depends

app = FastAPI()

async def get_token(x_token: str = Header(...)):
    """Verifica token — usata come dipendenza."""
    if x_token != "secret":
        raise HTTPException(status_code=403)
    return x_token

@app.get("/protetto")
def protetto(token: str = Depends(get_token)):
    return {"token": token}
```

## Dipendenza per sessione DB

`yield` trasforma la dipendenza in un context manager: FastAPI esegue il codice fino a `yield`, passa il valore (la sessione DB) all'endpoint, e quando l'endpoint termina (o fallisce), esegue il codice dopo `yield` (qui: `db.close()`). Questo garantisce che la connessione venga sempre chiusa, anche in caso di eccezione. È il pattern standard per DB in FastAPI.

```python
from sqlalchemy.orm import Session

def get_db():
    db = SessionLocal()
    try:
        yield db  # yield = context manager
    finally:
        db.close()

@app.get("/utenti/{id}")
def get_utente(id: int, db: Session = Depends(get_db)):
    return db.query(UtenteDB).filter(UtenteDB.id == id).first()
```

## Dipendenza con parametri e classi

```python
from fastapi import Depends, Query

class Paginazione:
    def __init__(self, skip: int = Query(0, ge=0), limit: int = Query(10, ge=1)):
        self.skip = skip
        self.limit = limit

@app.get("/items")
def list_items(pag: Paginazione = Depends()):
    """FastAPI istanzia Paginazione automaticamente."""
    return {"skip": pag.skip, "limit": pag.limit}
```

## Dipendenze annidate

```python
def verify_token(x_token: str = Header(...)):
    if x_token != "secret":
        raise HTTPException(status_code=403)
    return x_token

def verify_key(x_key: str = Header(...)):
    if x_key != "key":
        raise HTTPException(status_code=403)
    return x_key

# Dipendenza che usa altre dipendenze
def verify_auth(
    token: str = Depends(verify_token),
    key: str = Depends(verify_key),
):
    return {"token": token, "key": key}

@app.get("/secure")
def secure(auth: dict = Depends(verify_auth)):
    return {"status": "ok", **auth}
```

## yield — contesto e cleanup

```python
async def get_session():
    session = Session()
    try:
        yield session
    finally:
        session.close()

@app.get("/query")
def query(db: Session = Depends(get_session)):
    return db.query(...)
```

## Dipendenze globali

`dependencies=[Depends(...)]` nel costruttore di `FastAPI()` o `APIRouter()` applica la dipendenza a TUTTI gli endpoint del router/app. Utile per autenticazione globale, logging, rate limiting. Attenzione: la dipendenza globale viene eseguita anche per route che potrebbero non averne bisogno — se solo pochi endpoint richiedono autenticazione, usa `Depends()` direttamente su quelli.

```python
app = FastAPI(dependencies=[Depends(verify_token)])

# O per router
router = APIRouter(dependencies=[Depends(verify_token)])
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Depends()` senza argomento | Manca la funzione dipendenza | Passa sempre una callable a `Depends(nome_func)` |
| Dipendenza eseguita ma non iniettata | Parametro non chiamato come la dipendenza | Il nome del parametro NON conta — conta solo `Depends(func)` |
| `yield` senza `try/finally` | Risorsa (es. DB) non chiusa se eccezione | Usa sempre `try: yield x; finally: cleanup` |
| Dipendenza globale blocca endpoint pubblici | Login page richiede token | Non usare dipendenze globali per auth se hai endpoint pubblici |

## Best practice

- **Dipendenze piccole e focalizzate**: una funzione = una responsabilità (es. `get_db`, `verify_token`, `pagination`)
- **Override per test**: FastAPI permette di sostituire dipendenze in testing via `app.dependency_overrides[func] = mock_func`
- **Classi come dipendenza**: se la dipendenza ha stato o configurazione, usa una classe con `__init__` — FastAPI la istanzia automaticamente
- **Gerarchia naturale**: una dipendenza può usare altre dipendenze (es. `verify_auth` usa `verify_token` + `verify_key`) — FastAPI risolve l'albero automaticamente
