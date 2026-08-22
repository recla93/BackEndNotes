---
topic: "Testing e Deploy — FastAPI"
tags: [python, fastapi, testing, deploy, pytest]
nav_prev: "[[Auth e Sicurezza.md]]"
---
Riferimento ufficiale: [fastapi.tiangolo.com/tutorial/testing](https://fastapi.tiangolo.com/tutorial/testing/)

FastAPI usa `TestClient` (basato su `httpx`) per test HTTP senza server live. Il sistema DI permette di **override** delle dipendenze per isolare i test.

Vedi anche: [[BE-NOTES/Python/Strumenti/pytest|pytest]], [[BE-NOTES/Python/FastAPI/Dependency Injection|DI]] per override, [[BE-NOTES/Python/Strumenti/Docker per Python|Docker]] per deploy containerizzato.

`TestClient` (da `httpx`) simula richieste HTTP senza avviare un server reale. Ogni chiamata `client.get("/")` costruisce una request ASGI, la processa attraverso l'app FastAPI (con tutti i middleware e dipendenze), e restituisce una `Response` su cui puoi fare assert. Non c'è bisogno di server in ascolto — il test è completamente isolato e veloce. `response.json()` deserializza automaticamente il body JSON.

```python
# test_main.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"messaggio": "Benvenuto!"}

def test_crea_persona():
    response = client.post("/persone", json={
        "nome": "Mario",
        "eta": 25,
    })
    assert response.status_code == 200
    data = response.json()
    assert data["nome"] == "Mario"
```

## Test con database

`app.dependency_overrides` è il meccanismo chiave: sostituisce la dipendenza `get_db` con una versione che usa un database separato (in-memory o file temporaneo). In questo modo i test non toccano il DB di sviluppo/produzione. `Base.metadata.create_all(bind=engine)` crea le tabelle nel DB di test prima di eseguire i test.

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from main import app, get_db
from models import Base

# Database in-memory per test
TEST_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def override_get_db():
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db
Base.metadata.create_all(bind=engine)
```

## Eseguire

```bash
# Con pytest
pip install pytest httpx
pytest test_main.py -v

# Con coverage
pip install pytest-cov
pytest --cov=app test_main.py
```

## Deploy — Uvicorn + Gunicorn

```bash
# Sviluppo
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Produzione (con Gunicorn + Uvicorn workers)
pip install gunicorn
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

## Docker

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Environment-specific config

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str = "sqlite:///./dev.db"
    secret_key: str = "dev-key"
    debug: bool = True

    class Config:
        env_file = ".env"

settings = Settings()
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `AssertionError` su status code | Endpoint non trovato (404) o validazione fallita (422) | Controlla path e body della request di test |
| DB test influenzato da test precedenti | Dati residui tra test | Usa fixture con setup/teardown o database temporaneo per test |
| `override` non funziona | `dependency_overrides` impostato dopo aver creato il client | Imposta override PRIMA di creare `TestClient(app)` |
| Gunicorn non trova l'app | `main:app` sbagliato | Controlla che il modulo `main` esista e abbia la variabile `app` |
| `ModuleNotFoundError` in produzione | requirements.txt non include tutte le dipendenze | Usa `pip freeze > requirements.txt` o Poetry/UV |

## Best practice

- **Test isolati**: ogni test parte con DB pulito (usa fixture `autouse` o `yield_fixture` con setup/teardown)
- **Override nelle fixture**: usa `app.dependency_overrides[dep] = mock` in una fixture `pytest` di setup
- **Test client come fixture**: crea `client = TestClient(app)` una volta e riutilizzalo in tutti i test
- **Config via env**: mai hardcodare `DATABASE_URL` o `SECRET_KEY` — usa `pydantic-settings` con `.env`
- **Docker multistage**: separa build e runtime per immagini più piccole e sicure
