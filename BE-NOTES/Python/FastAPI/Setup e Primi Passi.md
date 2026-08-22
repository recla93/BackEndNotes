---
topic: "FastAPI — Setup e Primi Passi"
tags: [python, fastapi, backend, api, web]
nav_next: "[[Pydantic e Validazione.md]]"
---
Riferimento ufficiale: [fastapi.tiangolo.com/tutorial/first-steps](https://fastapi.tiangolo.com/tutorial/first-steps/)

FastAPI (2018) è il framework REST più moderno per Python. Basato su **Starlette** (ASGI) e **Pydantic** (validazione). Vantaggi rispetto a Flask/Django REST:
- **Performance ASGI** (comparabile a Node.js/Go)
- **Validazione automatica** con Pydantic
- **Documentazione OpenAPI** generata automaticamente
- **Async nativo** ma supporta sync
- **Type hint-driven** — meno boilerplate

Vedi anche: [[BE-NOTES/Python/FastAPI/Pydantic e Validazione|Pydantic]], [[BE-NOTES/Python/FastAPI/Dependency Injection|DI]], [[BE-NOTES/Python/FastAPI/Router e Middleware|Router]], [[BE-NOTES/Python/FastAPI/Database e SQLAlchemy|SQLAlchemy]], [[BE-NOTES/Python/Tecnologie/Async e AsyncIO|AsyncIO]].

```bash
pip install fastapi uvicorn
pip install sqlalchemy pydantic requests
```

## Prima App

Quando chiami `FastAPI()` viene creata un'istanza ASGI che tiene una tabella interna di route. Il decoratore `@app.get("/")` registra la funzione `root()` in quella tabella, associandola al metodo HTTP `GET` e al path `/`. A runtime, quando arriva una richiesta `GET /`, FastAPI cerca la route nella tabella, esegue la funzione e serializza automaticamente il dict restituito in JSON con status 200. Se la funzione non ha `return`, FastAPI risponde con 500.

```python
# main.py
from fastapi import FastAPI

app = FastAPI(title="My API", version="1.0.0")

@app.get("/")
def root():
    return {"messaggio": "Benvenuto!"}
```

## Avvio

Uvicorn è il server ASGI. `main:app` significa "modulo `main.py`, variabile `app` al suo interno". `--reload` fa ripartire il server a ogni modifica del codice (solo sviluppo). Apri `http://127.0.0.1:8000/docs` per la UI Swagger generata automaticamente.

```bash
uvicorn main:app --reload
# main: nome file (main.py)
# app: variabile FastAPI
# --reload: auto-restart a ogni modifica (solo sviluppo)
```

**Errore comune**: se uvicorn dà `ModuleNotFoundError`, non sei nella directory giusta o il file si chiama diversamente.

## Endpoint Base

FastAPI mappa i verbi HTTP decorando le funzioni. I path parameter (es. `{id}`) sono dichiarati come parametri funzione con lo stesso nome. Se un parametro ha un tipo Pydantic (es. `persona: Persona`), FastAPI legge il body JSON, lo valida contro lo schema, e passa direttamente l'oggetto validato.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Persona(BaseModel):
    nome: str
    eta: int
    città: str = "Roma"

# GET
@app.get("/")
def root():
    return {"messaggio": "Benvenuto!"}
@app.get("/persone/{id}")
def get_persona(id: int):
    return {"id": id, "nome": "Mario", "eta": 25}

# POST — body automaticamente validato con Pydantic
@app.post("/persone")
def crea_persona(persona: Persona):
    return {"messaggio": "Persona creata", "data": persona}

# PUT
@app.put("/persone/{id}")
def aggiorna_persona(id: int, persona: Persona):
    return {"id": id, "messaggio": "Aggiornata", "data": persona}

# DELETE
@app.delete("/persone/{id}")
def elimina_persona(id: int):
    return {"messaggio": f"Persona {id} eliminata"}
```

**Cosa succede per `POST /persone` con body `{"nome":"Anna","eta":30}`**: FastAPI usa `Persona.model_validate(json_data)` (Pydantic v2). Se `città` manca, usa il default. Se `eta` non è int, risponde 422. Se tutto ok, passa l'istanza validata alla funzione.

**Errore comune**: passare `id` non numerico (`/persone/abc`) — FastAPI risponde 422, non 404, perché fallisce la validazione del path parameter.

## Path vs Query Parameters

FastAPI distingue automaticamente: se il nome del parametro compare nella stringa del decoratore (es. `{user_id}`), è path parameter. Altrimenti è query parameter (letto dalla query string). Path parameter identifica una risorsa specifica; query parameter filtra/parametriza la risposta.

```python
# Path parameter — parte del URL path
@app.get("/utenti/{user_id}")
def get_utente(user_id: int): ...

# Query parameter — dopo ? nel URL
@app.get("/search")
def search(q: str, skip: int = 0, limit: int = 10): ...
```

**Errore comune**: endpoint su `/docs` sovrascrive la UI Swagger — usa `docs_url="/api/docs"`.

## Best practice

- **Type hint sempre**: senza type hint FastAPI non valida né documenta
- **Path per risorse, query per filtri**: `/utenti/1` non `/utente?id=1`
- **Separa le route**: usa `APIRouter` (vedi [[BE-NOTES/Python/FastAPI/Router e Middleware|Router]])
- **Async per I/O, sync per CPU**: `async def` per DB/HTTP, `def` per calcoli
