---
topic: "Moduli e Pacchetti — Python"
tags: [python, base, modules, packages, project-structure]
nav_prev: "[[Errori e Eccezioni.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/modules.html](https://docs.python.org/3/tutorial/modules.html)

Ogni file `.py` è un **modulo**; una cartella con `__init__.py` è un **pacchetto**. Python importa moduli con `import` e li cerca in `sys.path`.

A differenza di Java (package = directory, classpath), Python ha un sistema più flessibile ma attento a **circular imports** e **namespace pollution**.

Vedi anche: [[BE-NOTES/Python/FastAPI/Setup e Primi Passi|Struttura progetto FastAPI]], [[BE-NOTES/Python/Strumenti/Poetry e UV|Poetry/UV]] per gestione dipendenze.

## Import

```python
# Forme di import
import math
from math import sqrt, pi
from math import *  # sconsigliato (inquinamento namespace)
from pathlib import Path as Percorso  # alias

math.sqrt(16)   # 4.0
sqrt(16)        # 4.0 (importata direttamente)
```

## `__name__` e `__main__` — il Guard Pattern

In Java, ogni classe può avere un `main()` che parte se lanciata da CLI, ma non c'è un meccanismo built-in per sapere se è il punto d'ingresso. Python risolve con una variabile automatica: `__name__`.

**Come funziona**: ogni modulo ha `__name__`. Se eseguito direttamente via `python file.py`, `__name__` vale `"__main__"`. Se importato, `__name__` vale il nome del modulo. Questo permette di scrivere codice che funge sia da libreria sia da script.

```python
# file.py
def funzione():
    return "utile sia come libreria che come script"

if __name__ == "__main__":
    # Questo blocco parte SOLO se eseguito direttamente
    print(funzione())
```

Senza il guard: se importi `file` da un altro modulo, tutto il codice al module-level viene eseguito — anche effetti collaterali come print, connessioni DB, avvio server.

Il Guard Pattern è **obbligatorio** in qualsiasi script Python che venga importato.

## `sys.path` — dove Python cerca i moduli

A differenza di Java (classpath), Python cerca in `sys.path`, una lista che include:
1. La directory del file corrente
2. Directory in `PYTHONPATH` (variabile d'ambiente)
3. Directory di installazione standard (site-packages)

```python
import sys
print(sys.path)  # dove Python cerca i moduli
sys.path.append("/mio/path")  # aggiungi manualmente (sconsigliato: usa PYTHONPATH)
```

## Circular imports — il problema e la soluzione

Python risolve gli import **a runtime** (non a compile-time come Java). Due moduli che si importano a vicenda causano `ImportError` perché quando A importa B, B prova a importare A che non ha ancora finito di caricarsi.

```python
# ⚠️ module_a.py
from module_b import funzione_b  # ImportError: cannot import name 'funzione_b'

# module_b.py
from module_a import ClasseA     # module_a non è ancora completamente caricato
```

**Soluzioni**:
1. **Ristrutturare** — separare modelli, servizi, API in layer (mai bidirezionali)
2. **Late import** — importare dentro la funzione (non module-level)
3. **Usare `import module`** invece di `from module import nome` — accedi via namespace dopo che il modulo è caricato

## Creare un pacchetto

```
mio_pacchetto/
├── __init__.py   # Rende la cartella un pacchetto (può essere vuoto)
├── modulo_a.py
└── modulo_b.py
```

```python
# __init__.py — può esporre selettivamente
from .modulo_a import funzione_utile
__all__ = ["funzione_utile", "ModuloB"]

# Import da pacchetto
from mio_pacchetto import funzione_utile
from mio_pacchetto.modulo_b import ModuloB
```

## pip e PyPI

```bash
# Installare pacchetti
pip install requests
pip install requests==2.31.0  # versione specifica

# Salvare dipendenze
pip freeze > requirements.txt

# Installare da requirements
pip install -r requirements.txt
```

### Security — dipendenze

- **Pin versioni** in requirements.txt / pyproject.toml (evita breaking changes automatici)
- **Usa `pip-audit`** o `safety` per vulnerabilità note: `pip install pip-audit && pip-audit`
- **Non installare pacchetti come root** (usa virtual env)
- **Verifica hash** con `--require-hashes` in pip per supply chain attack
- **PyPI non è vetting** — controlla repo, manutenzione, download count prima di usare

## pyproject.toml (standard moderno)

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "mio-progetto"
version = "0.1.0"
dependencies = [
    "fastapi>=0.100.0",
    "sqlalchemy>=2.0",
]
```

## Virtual environment

```bash
# Creare
python -m venv venv

# Attivare (Windows)
venv\Scripts\activate

# Attivare (Mac/Linux)
source venv/bin/activate

# Disattivare
deactivate
```

## Best practice struttura progetti

```
src/
└── myproject/
    ├── __init__.py
    ├── main.py
    ├── models/
    ├── services/
    └── utils/
tests/
pyproject.toml
```

- **Evita `__init__.py` con logica** — usalo solo per import selettivi
- **`__all__`** controlla cosa esporta `from modulo import *`
- **Evita circular imports** — separa modelli, servizi, API in layer distinti
- **Nessuna logica eseguibile al modulo-level** eccetto definizioni e costanti
- **Convenzione naming**: `_privato` (underscore) = interno al modulo, non importare (non è enforced, è convenzione PEP 8)
