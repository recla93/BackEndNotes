---
topic: "Thread e Multiprocessing — Python"
tags: [python, threading, multiprocessing, concurrency, parallelism]
nav_prev: "[[Async e AsyncIO.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/threading.html](https://docs.python.org/3/library/threading.html)

Python ha **due livelli** di parallelismo:
- **`threading`**: concorrenza I/O-bound (ma **GIL** limita il parallelismo CPU)
- **`multiprocessing`**: parallelismo CPU-bound (ogni processo ha GIL separato)
- **`concurrent.futures`**: astrazione unificata per entrambi

### Il GIL (Global Interpreter Lock) spiegato

Il GIL è un mutex in CPython che permette a **un solo thread** di eseguire bytecode Python alla volta. Perché esiste? La gestione della memoria di Python (reference counting) non è thread-safe — senza GIL, ogni modifica a un oggetto richiederebbe lock costosi, rallentando il single-thread.

**Conseguenze pratiche**:
- Thread CPU-bound (es. `for i in range(10**8): pass`) NON scala con più thread — anzi, peggiora per overhead di context switching
- Thread I/O-bound (es. `requests.get()`, `time.sleep()`) funziona bene perché il thread rilascia il GIL durante l'attesa I/O
- `multiprocessing` bypassa il GIL perché ogni processo ha il suo interprete Python separato
- `asyncio` non ha problema GIL perché è single-thread cooperativo

### Regola pratica

| Tipo di carico | Soluzione |
|---|---|
| I/O-bound (file, network, DB) | `asyncio` (prima scelta) o `threading` |
| CPU-bound (calcoli, ML, image processing) | `multiprocessing` o `concurrent.futures.ProcessPoolExecutor` |
| Misto | `asyncio` + `run_in_executor` |

Vedi anche: [[BE-NOTES/Python/Tecnologie/Async e AsyncIO|AsyncIO]] (alternativa moderna per I/O), [[BE-NOTES/Python/Funzionale/Concetti Base|FP]] per parallelismo funzionale.

```python
import threading
import time

def lavoro(id: int, delay: float):
    time.sleep(delay)
    print(f"Task {id} completato")

# Creare thread
threads = []
for i in range(3):
    t = threading.Thread(target=lavoro, args=(i, i))
    threads.append(t)
    t.start()

# Attendere completamento
for t in threads:
    t.join(timeout=5)  # timeout opzionale
```

## Thread safety — Lock

```python
from threading import Lock

contatore = 0
lock = Lock()

def incrementa():
    global contatore
    for _ in range(10000):
        with lock:  # sezione critica
            contatore += 1

# RLock: lock rientrante (stesso thread può riacquisire)
from threading import RLock

# Semaphore: limita accessi simultanei
from threading import Semaphore
pool = Semaphore(5)  # max 5 thread contemporanei
```

## multiprocessing — parallelismo CPU-bound

```python
import multiprocessing as mp

def calcola_quadrato(n: int) -> int:
    return n ** 2

# Pool di processi
with mp.Pool(processes=4) as pool:
    risultati = pool.map(calcola_quadrato, range(100))

# Process singolo
p = mp.Process(target=lavoro, args=(1, 2))
p.start()
p.join()

# Attenzione: ogni processo ha la propria memoria
# Comunicazione via Queue, Pipe o shared memory
```

## Inter-process communication

```python
# Queue
def produttore(q: mp.Queue):
    for i in range(5):
        q.put(i)

def consumatore(q: mp.Queue):
    while True:
        item = q.get()
        if item is None:  # poison pill
            break
        print(f"Ricevuto: {item}")

q = mp.Queue()
p1 = mp.Process(target=produttore, args=(q,))
p2 = mp.Process(target=consumatore, args=(q,))
p1.start(); p2.start()
p1.join()
q.put(None)  # segnala fine
p2.join()
```

## concurrent.futures (astrazione moderna)

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import urllib.request

URLS = [
    "https://python.org",
    "https://docs.python.org",
    "https://pypi.org",
]

def fetch(url: str) -> str:
    with urllib.request.urlopen(url) as resp:
        return url, len(resp.read())

# Thread pool (I/O-bound)
with ThreadPoolExecutor(max_workers=3) as ex:
    results = ex.map(fetch, URLS)

# Process pool (CPU-bound)
with ProcessPoolExecutor(max_workers=4) as ex:
    results = ex.map(calcola_quadrato, range(100))

# Future per controllo fine-grained
futures = [ex.submit(calcola_quadrato, n) for n in range(10)]
for f in futures:
    print(f.result(timeout=2))
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| CPU-bound con thread non scala | GIL impedisce parallelismo reale — thread competono per il lock | Usa `multiprocessing` o `ProcessPoolExecutor` |
| Race condition senza Lock | Due thread modificano stessa variabile senza mutua esclusione | Usa `with lock:` attorno alla sezione critica |
| Deadlock | Lock annidati acquisiti in ordine diverso dai thread | Acquisisci sempre i lock nello stesso ordine, o usa `RLock` |
| `PicklingError` in multiprocessing | Dato passato a `Pool.map()` non serializzabile | Usa tipi semplici o personalizza `__getstate__`/`__setstate__` |
| `ValueError: cannot join current thread` | `thread.join()` chiamato dal thread stesso | Chiama `join()` solo da un thread diverso |
| Processi orfani | Processo principale termina senza aspettare i figli | Usa `Pool` come context manager (`with mp.Pool() as pool:`) |

## Best practice

- **Prima scelta per I/O**: `asyncio` (leggero, no GIL contention). `threading` solo se devi usare librerie blocking non-async
- **CPU pesante**: `multiprocessing` o scendi a C con numpy/Cython
- **`concurrent.futures`** è di solito più pulito di `threading`/`multiprocessing` diretto
- **Evita stato condiviso** tra thread — usa message passing (Queue) come in Go
- **`multiprocessing` ha overhead di serializzazione**: pickle dei dati tra processi può essere lento
