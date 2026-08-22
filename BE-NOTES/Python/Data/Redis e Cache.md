---
topic: "Redis e Caching — Python"
tags: [python, redis, cache, performance, redis-py]
---
Riferimento ufficiale redis-py: [redis-py.readthedocs.io](https://redis-py.readthedocs.io/)

Redis e un datastore in-memory chiave-valore usato come cache, message broker e database temporaneo. In Python si usa principalmente tramite il client `redis-py` (o `redis[hiredis]` per performance superiori).

Il caching e l'uso piu comune di Redis in Python: memorizzare risultati costosi (query DB, calcoli, API esterne) per evitarne la ripetizione. Altre modalita: session store, code di messaggi (pub/sub), rate limiting, contatori atomici.

Vedi anche:
[[BE-NOTES/Python/Data/Database Connection|Database Connection]],
[[BE-NOTES/Python/FastAPI/Database e SQLAlchemy|Database e SQLAlchemy]],
[[BE-NOTES/Python/Strumenti/Docker per Python|Docker per Python]].

## Connessione Redis

```bash
pip install redis[hiredis]
```

```python
import redis

r = redis.Redis(host="localhost", port=6379, db=0, decode_responses=True)

pool = redis.ConnectionPool(host="localhost", port=6379, db=0)
r = redis.Redis(connection_pool=pool)

r = redis.from_url("redis://localhost:6379/0")
```

`decode_responses=True` restituisce stringhe Python invece di byte. `hiredis` e un parser C piu veloce. I ConnectionPool sono thread-safe e riutilizzabili.

## Operazioni base (stringhe)

```python
import redis
r = redis.Redis(decode_responses=True)

r.set("utente:1:nome", "Alice")
r.get("utente:1:nome")                # 'Alice'

r.setex("sessione:token123", 3600, "utente_42")
r.ttl("sessione:token123")

r.exists("utente:1:nome")
r.delete("utente:1:nome")

r.incr("visite")
```

`setex` imposta valore + TTL in un'unica operazione atomica. `incr` e thread-safe senza race condition.

## Liste, Set, Sorted Set

```python
import redis
r = redis.Redis(decode_responses=True)

r.lpush("coda", "task1", "task2")
r.rpop("coda")
r.llen("coda")

r.sadd("tags:articolo:42", "python", "redis", "tutorial")
r.smembers("tags:articolo:42")
r.sismember("tags:articolo:42", "python")

r.zadd("classifica", {"Alice": 100, "Bob": 85})
r.zrange("classifica", 0, -1, withscores=True, desc=True)
```

Liste Redis sono ottime per code di lavoro. Sorted set per classifiche. Set supportano operazioni insiemistiche lato server (`sinter`, `sunion`).

## Decoratore cache

```python
import redis
import functools
import json

r = redis.Redis(decode_responses=True)

def cache(ttl: int = 300):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = f"cache:{func.__name__}:{hash(str(args) + str(kwargs))}"
            cached = r.get(key)
            if cached is not None:
                return json.loads(cached)
            result = func(*args, **kwargs)
            r.setex(key, ttl, json.dumps(result))
            return result
        return wrapper
    return decorator

@cache(ttl=60)
def get_utente(user_id: int) -> dict:
    return {"id": user_id, "nome": "Alice"}
```

Cache-Aside pattern: prima controlla Redis, se manca calcola e salva. TTL fa scadere automaticamente.

## Pub/Sub (messaggistica)

```python
import redis, threading

r = redis.Redis(decode_responses=True)

def publish():
    r.publish("canale:notifiche", "Nuovo utente registrato!")

def subscribe():
    pubsub = r.pubsub()
    pubsub.subscribe("canale:notifiche")
    for message in pubsub.listen():
        if message["type"] == "message":
            print(f"Ricevuto: {message['data']}")

threading.Thread(target=subscribe, daemon=True).start()
publish()
```

Pub/Sub non e persistente: se non c'e subscriber, il messaggio e perso. Ideale per notifiche in tempo reale.

## Redis in FastAPI (async)

```python
import redis.asyncio as aioredis
from fastapi import FastAPI

app = FastAPI()
r = aioredis.from_url("redis://localhost:6379", decode_responses=True)

@app.on_event("startup")
async def startup():
    await r.ping()

@app.get("/cached-data")
async def get_cached():
    cached = await r.get("key:data")
    if cached:
        return {"data": cached}
    await r.setex("key:data", 60, "result")
    return {"data": "result"}
```

## Errori comuni

- **Dimenticare `decode_responses=True`**: `r.get()` restituisce `bytes` invece di `str`.
- **TTL troppo lungo**: cache stale per ore. Imposta TTL ragionevole.
- **Cache key collision**: usa namespace `utente:{id}:dettagli`.
- **Serializzazione manuale**: Redis salva solo stringhe/byte. Usa `json.dumps()`/`json.loads()`.
- **Usare Redis come database primario**: Redis non garantisce persistenza. E un cache/store temporaneo, non un database.

## Best Practices & Conventions

- Usa `decode_responses=True` per evitare byte ovunque.
- Namespace le chiavi con `:`, es. `utente:42:profile`, `cache:articoli:lista`.
- Imposta TTL su ogni chiave. Redis e in-memory: senza TTL la memoria si riempie.
- Per cache di funzioni, includi nome funzione + argomenti nella key per evitare collisioni.
- Usa ConnectionPool per applicazioni multi-thread.
- In produzione, non esporre Redis direttamente; usa un proxy o password. Redis non ha autenticazione robusta.
- redis-py e sincrono. Per asincrono (FastAPI) usa `redis.asyncio`. Per performance, installa `hiredis`.
