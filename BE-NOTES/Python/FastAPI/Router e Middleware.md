---
topic: "Router e Middleware — FastAPI"
tags: [python, fastapi, router, middleware]
nav_prev: "[[Dependency Injection.md]]"
nav_next: "[[Database e SQLAlchemy.md]]"
---
Riferimento ufficiale: [fastapi.tiangolo.com/tutorial/bigger-applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)

Per progetti oltre il file singolo, FastAPI fornisce `APIRouter` (come Blueprints in Flask). I **middleware** sono layer di processing request/response, come filtri in Spring o middleware in Express.

Vedi anche: [[BE-NOTES/Python/FastAPI/Setup e Primi Passi|Setup]], [[BE-NOTES/Python/FastAPI/Dependency Injection|DI]] per dipendenze a livello router.

`APIRouter` crea un gruppo di route isolato, con un proprio prefisso (`/utenti`) e tag per la documentazione. Ogni router può avere le proprie dipendenze e middleware. È l'equivalente dei Blueprints in Flask o dei module in Spring. I decoratori `@router.get(...)` funzionano come `@app.get(...)` ma registrano la route sul router invece che sull'app globale.

```python
# app/routers/utenti.py
from fastapi import APIRouter

router = APIRouter(prefix="/utenti", tags=["utenti"])

@router.get("/")
def lista_utenti():
    return [{"nome": "Mario"}, {"nome": "Luigi"}]

@router.get("/{id}")
def get_utente(id: int):
    return {"id": id, "nome": "Mario"}
```

## Includere router nell'app

```python
# app/main.py
from fastapi import FastAPI
from app.routers import utenti, prodotti

app = FastAPI()
app.include_router(utenti.router)
app.include_router(prodotti.router)

# Ora endpoint disponibili:
# GET /utenti
# GET /utenti/1
# GET /prodotti
```

## Middleware custom

Un middleware è una funzione che viene eseguita per OGNI richiesta, prima e dopo l'endpoint. Riceve la request e una callable `call_next` che passa la richiesta alla route (o al middleware successivo). Il codice prima di `call_next` è la fase di pre-processing; quello dopo è la fase di post-processing. In questo esempio, il middleware misura il tempo di risposta e aggiunge un header HTTP custom. I middleware si eseguono in ordine di registrazione.

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def log_tempo(request: Request, call_next):
    """Middleware: logga tempo di risposta."""
    start = time.time()
    response = await call_next(request)
    elapsed = time.time() - start
    response.headers["X-Elapsed"] = str(elapsed)
    print(f"{request.url.path} - {elapsed:.3f}s")
    return response
```

## Tag e metadata

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/items",
    tags=["items"],
    responses={404: {"description": "Non trovato"}},
)

# L'ordine conta per la documentazione OpenAPI
# I tag raggruppano gli endpoint nella UI /docs
```

## Esempio struttura progetto

```
app/
├── main.py           # FastAPI app, inclusioni, CORS
├── routers/
│   ├── __init__.py
│   ├── utenti.py
│   └── prodotti.py
├── models/
│   ├── __init__.py
│   └── database.py
├── schemas/
│   ├── __init__.py
│   └── pydantic.py
└── dependencies.py   # dipendenze condivise
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Endpoint non trovato (404) | Router non incluso in `app.include_router()` | Aggiungi `app.include_router(nome_router)` in main.py |
| Path duplicato tra router | Due router hanno stesso prefix + path | Usa prefissi diversi per ogni router |
| Middleware non eseguito | Middleware registrato dopo l'endpoint | Sposta middleware prima delle route |
| Middleware async in route sync | Middleware usa `async def` ma route è `def` | Compatibile — FastAPI gestisce la conversione |

## Best practice

- **Un file per router**: separa utenti, prodotti, ordini in file diversi
- **Prefisso logico**: il prefix del router rispecchia la risorsa (`/utenti`, `/prodotti`, `/ordini`)
- **Middleware leggeri**: non fare operazioni pesanti in middleware (bloccano tutte le request)
- **Dependencies a livello router**: se TUTTI gli endpoint del router servono auth, metti `dependencies=[Depends(verify_token)]` nel router, non globalmente
