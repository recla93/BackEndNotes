---
topic: "Async e AsyncIO — Python"
tags: [python, async, asyncio, concurrency]
nav_prev: "[[Generator e Iterator.md]]"
nav_next: "[[Thread e Multiprocessing.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/asyncio.html](https://docs.python.org/3/library/asyncio.html)

Async/await in Python (PEP 492, Python 3.5+) è la soluzione per **I/O-bound concorrenza** senza thread. L'event loop single-thread coordina coroutine che cedono il controllo volontariamente con `await`.

### Come funziona l'event loop

Immagina un **scheduler single-thread** che gestisce una coda di coroutine. Quando una coroutine fa `await`, dice "ehi, aspetto I/O, nel frattempo fai lavorare altri". L'event loop passa a un'altra coroutine pronta, e quando l'I/O è finito, riprende la prima. È **cooperativo** (le coroutine decidono quando cedere), non **preemptive** (come i thread dove il sistema operativo decide).

A differenza di threading:
- **Niente GIL contention** (cooperativo, non preemptive)
- **Overhead minimo** (migliaia di coroutine su 1 thread)
- **Non serve locking** (single-thread, no race condition su dati condivisi)

Adatto per: API server (FastAPI), web scraping, microservizi, proxy. **Non** per CPU-bound (usa [[BE-NOTES/Python/Tecnologie/Thread e Multiprocessing|multiprocessing]]).

Vedi anche: [[BE-NOTES/Python/Tecnologie/Generator e Iterator|Generator]] (base delle coroutine), [[BE-NOTES/Python/FastAPI/Setup e Primi Passi|FastAPI]] (async framework), [[BE-NOTES/Python/Data/Database Connection|Database]] per SQLAlchemy async.

### ⚠️ `asyncio.run()` crea un nuovo event loop

`asyncio.run()` crea un nuovo event loop ogni volta. Non chiamarlo dentro una coroutine già in esecuzione — usa `await` direttamente. In FastAPI, i path handler `async def` hanno già l'event loop gestito dal framework.

```python
import asyncio

async def saluta(nome: str) -> str:
    await asyncio.sleep(0.5)  # simulazione I/O
    return f"Ciao {nome}"

# Eseguire una coroutine
risultato = asyncio.run(saluta("Mario"))
```

## await e concorrenza

```python
async def fetch_dati(nome: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"Risultato da {nome}"

async def main():
    # Sequenziale (come sync)
    r1 = await fetch_dati("API 1", 2)
    r2 = await fetch_dati("API 2", 3)
    # Totale: ~5s

    # Concorrente — gather
    risultati = await asyncio.gather(
        fetch_dati("API 1", 2),
        fetch_dati("API 2", 3),
        fetch_dati("API 3", 1),
    )
    # Totale: ~3s (più lenta delle tre)

    # as_completed — risultati non appena pronti
    for cors in asyncio.as_completed([
        fetch_dati("API 1", 2),
        fetch_dati("API 2", 3),
    ]):
        ris = await cors
        print(f"Pronto: {ris}")

asyncio.run(main())
```

## Task — controllo avanzato

```python
async def main():
    # Creare task esplicitamente
    task = asyncio.create_task(fetch_dati("API", 2))

    # Attendere con timeout
    try:
        risultato = await asyncio.wait_for(task, timeout=1.0)
    except asyncio.TimeoutError:
        print("Timeout!")

    # Eseguire più task
    tasks = [
        asyncio.create_task(fetch_dati(f"API {i}", i))
        for i in range(3)
    ]
    risultati = await asyncio.gather(*tasks, return_exceptions=True)

asyncio.run(main())
```

## async/await vs threading vs multiprocessing

| Caratteristica | async/await | Threading | Multiprocessing |
|---|---|---|---|
| **Concorrenza** | ✅ I/O-bound | ✅ I/O-bound | ✅ CPU-bound |
| **GIL** | Non serve (cooperativo) | Conteso | Ogni processo ha il suo |
| **Overhead** | Molto basso | Medio | Alto |
| **Parallelo** | No (single-thread) | No (GIL) | Sì |
| **Complessità** | Media | Media | Alta |

## asyncio + subprocess

```python
import asyncio

async def esegui_comando(cmd: str) -> tuple[int, str]:
    proc = await asyncio.create_subprocess_shell(
        cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )
    stdout, stderr = await proc.communicate()
    return proc.returncode, stdout.decode()
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `RuntimeError: asyncio.run() cannot be called from a running event loop` | `asyncio.run()` chiamato dentro una coroutine | Usa `await` direttamente — `asyncio.run()` va solo nel `main()` entry point |
| Event loop bloccato | Chiamata bloccante sync (es. `time.sleep()`, `requests.get()`) dentro coroutine | Usa `await asyncio.sleep()`, `httpx.AsyncClient`, o `loop.run_in_executor()` |
| `gather` non raccoglie eccezioni | `return_exceptions=False` (default) — la prima eccezione blocca le altre | Usa `return_exceptions=True` e gestisci i risultati |
| `Task was destroyed but it is pending!` | Task non awaitato prima dello shutdown | Raccogli tutti i task: `tasks = asyncio.all_tasks()` e cancellali |
| Race condition su stato condiviso | Due coroutine modificano la stessa variabile | Usa `asyncio.Lock()` — anche se single-thread, await cede il controllo |

## Best practice asyncio

- Usare solo librerie **async-native** con `await` (non blocking)
- Per DB: `asyncpg`, `databases`, SQLAlchemy 2.0 async
- Per HTTP: `httpx.AsyncClient`, `aiohttp`
- Per file: `aiofiles`
- Non mischiare `asyncio.run()` con event loop già attivo
- Gestire timeout con `asyncio.wait_for()`
