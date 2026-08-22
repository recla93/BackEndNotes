---
topic: "File e I/O — Python"
tags: [python, base, file-io, filesystem]
nav_prev: "[[Liste Tuple Set Dict.md]]"
nav_next: "[[Errori e Eccezioni.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/inputoutput.html](https://docs.python.org/3/tutorial/inputoutput.html)

Python offre diversi livelli per operazioni su file: da `open()` classico a `pathlib` (moderno, preferito), a `csv`, `json` per formati strutturati.

**Regola d'oro**: usa sempre `with` (context manager) per garantire chiusura automatica del file, anche in caso di eccezioni.

Vedi anche: [[BE-NOTES/Python/Core Concepts/Errori e Eccezioni|Errori]] per gestione eccezioni I/O, [[BE-NOTES/Python/Strumenti/Docker per Python|Docker]] per volumi e persistenza.

```python
# Leggere tutto
with open("testo.txt", "r", encoding="utf-8") as f:
    contenuto = f.read()

# Leggere riga per riga
with open("testo.txt", "r", encoding="utf-8") as f:
    for riga in f:
        print(riga.strip())

# Leggere tutte le righe in lista
with open("testo.txt") as f:
    righe = f.readlines()

# Scrivere
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Prima riga\n")
    f.write("Seconda riga\n")

# Append
with open("output.txt", "a", encoding="utf-8") as f:
    f.write("Terza riga\n")

# Il context manager (with) chiude automaticamente il file
# anche in caso di eccezioni
```

## JSON

```python
import json

# Lettura
with open("dati.json", "r", encoding="utf-8") as f:
    dati = json.load(f)  # dict/list Python

# Scrittura
dati = {"nome": "Mario", "eta": 25}
with open("dati.json", "w", encoding="utf-8") as f:
    json.dump(dati, f, indent=2, ensure_ascii=False)
```

## pathlib (moderno, preferito a os.path) — spiegazione

Prima di Python 3.4, i path erano stringhe manipolate con `os.path.join()`, `os.path.exists()`, ecc. Era facile sbagliare:
```python
# Vecchio stile — stringhe + os.path
percorso = os.path.join("cartella", "sotto", "file.txt")  # attenzione: \ o /?
if os.path.exists(percorso):
    with open(percorso) as f: ...
```

### Cosa fa pathlib
`pathlib` (3.4+) tratta i path come oggetti, non stringhe:
```python
from pathlib import Path

percorso = Path("cartella") / "sotto" / "file.txt"  # usa / come separatore!
# Sei su Windows: Path("cartella\\sotto\\file.txt")
# Sei su Linux:   Path("cartella/sotto/file.txt")
# Stesso codice!

if percorso.exists():
    contenuto = percorso.read_text()  # no open/close necessario
```

### Vantaggi concreti
- **Cross-platform automatico**: `Path("a") / "b"` produce `a\b` su Windows, `a/b` su Linux
- **Operazioni comode**: `path.read_text()`, `path.write_text()`, `path.glob("*.py")`
- **Metodi incapsulati**: `path.suffix` (`.txt`), `path.stem` (`file`), `path.parent` (la cartella)
- **Più leggibile**: niente `os.path.join(os.path.dirname(...), ...)`

```python
from pathlib import Path

base = Path("cartella")
base.mkdir(exist_ok=True)  # crea cartella

# Glob
list(base.glob("*.txt"))      # tutti .txt in cartella
list(base.rglob("*.py"))      # ricorsivamente

# Verifiche
base.exists()    # True/False
base.is_file()   # True/False
base.is_dir()    # True/False

# Path assoluto
base.resolve()  # C:\...\cartella
base.name       # "cartella"
base.suffix     # ".txt"
base.stem       # nome senza suffisso

# Lettura/scrittura comoda
Path("file.txt").write_text("contenuto", encoding="utf-8")
contenuto = Path("file.txt").read_text(encoding="utf-8")
```

## CSV

```python
import csv

# Lettura
with open("dati.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)  # ogni riga è un dict
    for riga in reader:
        print(riga["nome"], riga["eta"])

# Scrittura
with open("dati.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["nome", "eta"])
    writer.writeheader()
    writer.writerow({"nome": "Mario", "eta": 25})
```
