---
topic: "Auth e Sicurezza — FastAPI"
tags: [python, fastapi, auth, jwt, security]
nav_prev: "[[Database e SQLAlchemy.md]]"
nav_next: "[[Testing e Deploy.md]]"
---
Riferimento ufficiale: [fastapi.tiangolo.com/tutorial/security](https://fastapi.tiangolo.com/tutorial/security/)

FastAPI integra `OAuth2PasswordBearer` e `OAuth2PasswordRequestForm` per autenticazione. Pattern standard: JWT Bearer token + password hashing (bcrypt).

A differenza di Django (auth tutto inclusivo), FastAPI è **agnostico** — fornisce gli strumenti di base, la logica di autenticazione è tua.

Vedi anche: [[BE-NOTES/Python/FastAPI/Dependency Injection|DI]] per proteggere route, [[BE-NOTES/Python/FastAPI/Testing e Deploy|Testing]] per testare endpoint protetti.

`OAuth2PasswordBearer` dice a FastAPI: "leggi il token dal header `Authorization: Bearer <token>`". Se il token manca, risponde 401 automaticamente senza eseguire la route. `OAuth2PasswordRequestForm` è una dipendenza built-in che estrae `username` e `password` da un form POST. La funzione `get_current_user` è una dipendenza protettiva: ogni route che la usa come `Depends(get_current_user)` richiede un token valido.

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# Fake DB
UTENTI = {
    "mario": {"username": "mario", "password": "secret123"},
}

def get_current_user(token: str = Depends(oauth2_scheme)):
    """Decodifica token e restituisce utente."""
    user = fake_decode_token(token)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@app.post("/token")
def login(form: OAuth2PasswordRequestForm = Depends()):
    """Restituisce token JWT."""
    user = UTENTI.get(form.username)
    if not user or user["password"] != form.password:
        raise HTTPException(status_code=401, detail="Credenziali errate")
    token = create_access_token(data={"sub": user["username"]})
    return {"access_token": token, "token_type": "bearer"}

@app.get("/users/me")
def read_users_me(current_user = Depends(get_current_user)):
    return current_user
```

## JWT (JSON Web Token)

JWT è un token firmato digitalmente. `jwt.encode()` crea un payload JSON (es. `{"sub": "mario", "exp": 1700000000}`), lo firma con `SECRET_KEY` usando l'algoritmo `HS256`, e restituisce una stringa. `jwt.decode()` verifica la firma e la scadenza (`exp`): se la firma non matcha o il token è scaduto, lancia `PyJWTError`. `CryptContext` con `bcrypt` gestisce l'hashing delle password: `hash_password()` produce hash diversi ogni volta (salt automatico), `verify_password()` controlla se la password in chiaro matcha l'hash.

```python
from datetime import datetime, timedelta, timezone
import jwt
from passlib.context import CryptContext

SECRET_KEY = "chiave-super-segreta"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict | None:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except jwt.PyJWTError:
        return None
```

## CORS

CORS (Cross-Origin Resource Sharing) è un meccanismo di sicurezza del browser che blocca richieste da domini diversi. Il middleware CORS di FastAPI aggiunge gli header HTTP necessari (`Access-Control-Allow-Origin`, ecc.) alle risposte. `allow_origins` elenca i domini autorizzati (es. il frontend Angular su `localhost:4200`). `allow_credentials=True` permette l'invio di cookie. Attenzione: `allow_origins=["*"]` abilita TUTTI i domini — va bene solo in sviluppo, mai in produzione.

```python
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],  # frontend
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `401 Unauthorized` su route protetta | Token mancante o scaduto | Controlla header `Authorization: Bearer <token>` |
| `403 Forbidden` | Token valido ma utente non autorizzato | Aggiungi controllo ruoli/permessi nella dipendenza |
| CORS blocca richieste | `allow_origins` non include il dominio frontend | Aggiungi l'origine esatta del frontend |
| JWT decodifica fallisce silenziosamente | `decode_token()` restituisce `None` senza errore | Logga il tipo di eccezione per debugging |

## Best practice sicurezza

- Password sempre hashate (bcrypt/argon2)
- JWT con scadenza (15-30 min)
- HTTPS in produzione
- Validazione input (Pydantic)
- Rate limiting (slowapi)
- CSRF per cookie-based auth
- Secrets in variabili d'ambiente, mai nel codice
