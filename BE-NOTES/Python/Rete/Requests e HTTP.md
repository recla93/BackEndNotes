---
topic: "Requests e HTTP — Python"
tags: [python, http, requests, httpx, api, rest]
---
Riferimento ufficiale requests: [requests.readthedocs.io](https://requests.readthedocs.io/)
Riferimento ufficiale httpx: [www.python-httpx.org](https://www.python-httpx.org/)

`requests` e la libreria standard de facto per HTTP in Python: semplice, leggibile, ricca di funzionalita. `httpx` e l'evoluzione moderna con supporto nativo async e HTTP/2.

Entrambe gestiscono GET, POST, PUT, DELETE, sessioni, header, parametri, file upload, autenticazione e gestione errori. `httpx` offre una API identica a `requests` ma con `async` e `httpx.Client()` piu performante.

Vedi anche:
[[BE-NOTES/Python/FastAPI/Setup e Primi Passi|FastAPI]],
[[BE-NOTES/Python/Tecnologie/Async e AsyncIO|Async e AsyncIO]],
[[BE-NOTES/Python/Data/Database Connection|Database Connection]].

## Requests base (GET e POST)

```python
import requests

# GET
response = requests.get(
    "https://api.github.com/users/octocat",
    params={"per_page": 10},
    timeout=5,
)

print(response.status_code)   # 200
print(response.json())         # dict dalla risposta JSON
print(response.headers)        # dict degli header

# POST con JSON
payload = {"nome": "Alice", "email": "alice@example.com"}
response = requests.post(
    "https://api.example.com/users",
    json=payload,
    headers={"Authorization": "Bearer token123"},
    timeout=5,
)
```

`.json()` restituisce un dict dal body JSON. `.text` restituisce il body come stringa. `.content` restituisce bytes. `.raise_for_status()` solleva eccezione per codici 4xx/5xx.

## Gestione errori

```python
import requests
from requests.exceptions import RequestException, Timeout, HTTPError

try:
    response = requests.get("https://api.example.com/data", timeout=3)
    response.raise_for_status()
except Timeout:
    print("Richiesta scaduta")
except HTTPError as e:
    print(f"Errore HTTP: {e.response.status_code}")
except RequestException as e:
    print(f"Errore di rete: {e}")
```

`raise_for_status()` trasforma codici 4xx/5xx in eccezioni. `Timeout` per richieste lente. `ConnectionError` per problemi di rete. Cattura sempre `RequestException` come fallback.

## Sessioni (connessione riutilizzata)

```python
import requests

with requests.Session() as session:
    session.headers.update({"Authorization": "Bearer token123"})

    # La sessione riutilizza la connessione e mantiene cookie/header
    r1 = session.get("https://api.example.com/users")
    r2 = session.get("https://api.example.com/users/me")

# All'uscita dal with, la sessione viene chiusa
```

Le sessioni riutilizzano la connessione TCP (keep-alive), riducendo latenza. Mantengono cookie e header tra richieste. Usa sempre `with` per garantire la chiusura.

## File upload

```python
import requests

# File
with open("foto.jpg", "rb") as f:
    response = requests.post(
        "https://api.example.com/upload",
        files={"file": f},
        timeout=30,
    )

# Multipart con metadati
response = requests.post(
    "https://api.example.com/upload",
    files={"file": ("foto.jpg", open("foto.jpg", "rb"), "image/jpeg")},
    data={"descrizione": "Foto profilo"},
)
```

`files` accetta tuple `(nome_file, file_object, content_type)` per controllo fine. Usa `timeout` alto per upload. Per file grandi, considera `requests-toolbelt` per streaming.

## Httpx (sincrono e async)

```python
import httpx

# Sincrono (stile requests)
with httpx.Client() as client:
    response = client.get("https://api.example.com/data", timeout=5)
    print(response.json())

# Async
import asyncio

async def fetch_all():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
        return response.json()

data = asyncio.run(fetch_all())
```

`httpx` e night-time compatibile con `requests` nell'API base. `httpx.Client()` e intrinsecamente piu performante. `AsyncClient` e essenziale per app asincrone (FastAPI, aiohttp).

## Httpx — richieste parallele

```python
import asyncio
import httpx

async def fetch_parallel():
    urls = [
        "https://api.example.com/users",
        "https://api.example.com/posts",
        "https://api.example.com/comments",
    ]

    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]

results = asyncio.run(fetch_parallel())
```

`asyncio.gather()` esegue le richieste in parallelo. Decine di richieste simultanee senza bloccare il thread. Gestisci i timeout singolarmente su ogni richiesta.

## Httpx vs Requests

| Caratteristica | requests | httpx |
|---------------|----------|-------|
| Stdlib | No | No |
| Async | No | Si (AsyncClient) |
| HTTP/2 | No | Si |
| Performante | Si | Piu (Client nativo) |
| API | Standard | Compatibile |
| Dipendenze | urllib3 | httpcore |
| Quando usare | Script semplici, legacy | Nuovi progetti, async |

## Errori comuni

- **Dimenticare `timeout`**: richieste che rimangono in attesa per minuti. Imposta sempre `timeout`.
- **Non chiamare `raise_for_status()`**: 4xx/5xx vengono silenziosamente ignorati.
- **Sessioni non chiuse**: connessioni aperte. Usa `with requests.Session()`.
- **`.json()` su body non JSON**: `ValueError` se la risposta non e JSON. Controlla `response.headers['Content-Type']`.
- **URL senza schema**: `requests.get("api.example.com")` fallisce. Aggiungi `https://`.
- **httpx sync vs async**: chiamare `httpx.get()` (senza Client) e piu lento. Usa sempre `Client()` o `AsyncClient()`.

## Best Practices & Conventions

- Imposta sempre **`timeout`** su ogni richiesta. 5-10 secondi e un buon default.
- Usa **`raise_for_status()`** per esporre errori HTTP.
- Riusa la connessione con **`Session()`** o **`Client()`** per richieste multiple.
- Per nuovi progetti, scegli **httpx**: supporta async ed HTTP/2.
- Per script semplici, **requests** va benissimo.
- In app asincrone (FastAPI), usa sempre **`httpx.AsyncClient`** per non bloccare l'event loop.
- Usa **`resp.json()`** solo se sei sicuro che il Content-Type sia JSON.
