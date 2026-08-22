---
topic: "Context Manager — Python"
tags: [python, context-manager, with, resources]
nav_prev: "[[Type Hints.md]]"
nav_next: "[[Generator e Iterator.md]]"
---
Riferimento ufficiale: [docs.python.org/3/reference/compound_stmts.html#the-with-statement](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement)

## Cosa sono

Permettono di gestire risorse (file, connessioni, lock) con
setup e cleanup automatici tramite il costrutto `with`.

**Il problema che risolvono**: in Java, gestisci risorse con `try-with-resources` o `try {} finally {}`. In Python, `with` è syntactic sugar per lo stesso pattern — ma più elegante e garantito (anche con eccezioni intermedie).

Python li usa ovunque: `with open()`, `with lock:`, `with Session()`, `with transaction:`. Se una classe gestisce risorse che devono essere rilasciate, dovrebbe essere un context manager.

Vedi anche: [[BE-NOTES/Python/Core Concepts/File e IO|File I/O]], [[BE-NOTES/Python/OOP/Magic Methods|Magic Methods]] per `__enter__/__exit__`, [[BE-NOTES/Python/FastAPI/Dependency Injection|FastAPI Depends]] per `yield` come context manager.

```python
# Esempio classico: file
with open("file.txt", "r") as f:
    contenuto = f.read()
# Il file è chiuso automaticamente anche in caso di eccezioni
```

## Implementazione con classe (__enter__ / __exit__)

```python
class ConnessioneDB:
    def __enter__(self):
        """Setup: apre connessione."""
        print("Apro connessione...")
        return self  # l'oggetto assegnato a "as"

    def __exit__(self, exc_type, exc_val, exc_tb):
        """Cleanup: chiude connessione."""
        print("Chiudo connessione...")
        # Se vuoi sopprimere eccezione: return True
        # Se vuoi propagare: return None/False (default)
        return False

    def esegui(self, query: str):
        print(f"Eseguo: {query}")

with ConnessioneDB() as db:
    db.esegui("SELECT * FROM utenti")
```

## Implementazione con contextlib

```python
from contextlib import contextmanager

@contextmanager
def temporaneo(nome_file: str, contenuto: str):
    """Crea file temporaneo e lo elimina dopo l'uso."""
    # Setup (equivalente a __enter__)
    with open(nome_file, "w") as f:
        f.write(contenuto)
    
    try:
        yield nome_file  # passa il valore a "as"
    finally:
        # Cleanup (equivalente a __exit__)
        import os
        os.remove(nome_file)

with temporaneo("test.txt", "Ciao!") as path:
    print(f"File {path} creato")
# File eliminato qui
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Risorsa non rilasciata su eccezione | `__exit__` non gestisce eccezione o manca `finally` in `@contextmanager` | Usa `try: yield finally:` — garantito anche con eccezioni |
| Eccezioni silenziosamente soppresse | `__exit__` restituisce `True` | Restituisci `None`/`False` per propagare eccezioni (default) |
| `@contextmanager` senza `yield` | Funzione non ritorna un generatore — `@contextmanager` si aspetta `yield` | Aggiungi `yield valore` dopo il setup |
| `ExitStack` non gestisce rollback parziale | Alcuni context manager entrati, eccezione prima di finire | `ExitStack` già chiude tutto in ordine inverso — ma assicurati che ogni `enter_context` sia dentro il `with` |
| `contextmanager` con generator esaurito | Riutilizzo di un context manager decorato | I `@contextmanager` sono monouso — creane uno nuovo per ogni `with` |

## contextlib utile

```python
from contextlib import suppress, redirect_stdout, nullcontext

# suppress — ignora eccezioni specifiche
with suppress(FileNotFoundError):
    os.remove("file.txt")

# redirect_stdout — cattura print
with open("log.txt", "w") as f:
    with redirect_stdout(f):
        print("Questo finisce nel file")

# ExitStack — multipli context manager dinamici
from contextlib import ExitStack

def gestisci_file(file_paths: list[str]):
    with ExitStack() as stack:
        files = [stack.enter_context(open(p)) for p in file_paths]
        # Lavora con i file...
        # ExitStack li chiude tutti nell'ordine inverso

# nullcontext — quando non serve contesto reale
from contextlib import nullcontext

def processa(file_path: str | None = None):
    # Se file_path è None, non apre file (no-op)
    ctx = open(file_path) if file_path else nullcontext()
    with ctx as risorsa:
        # risorsa è None se file_path è None
        ...
```
