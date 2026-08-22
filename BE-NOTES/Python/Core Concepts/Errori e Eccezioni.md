---
topic: "Errori e Eccezioni — Python"
tags: [python, base, errors, exceptions]
nav_prev: "[[File e IO.md]]"
nav_next: "[[Moduli e Pacchetti.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/errors.html](https://docs.python.org/3/tutorial/errors.html)

Python usa eccezioni per segnalare errori (come Java/C#), ma con una filosofia diversa: **EAFP** ("Easier to Ask for Forgiveness than Permission") — provi l'operazione e gestisci l'eccezione se fallisce, invece di controllare prima (LBYL).

Pattern comune vs Java: in Java controlli `if (file.exists())`, in Python fai `try: open(file)` e gestisci `FileNotFoundError`.

Vedi anche: [[BE-NOTES/Python/Core Concepts/Funzioni|Funzioni]] per gestione errori in funzioni, [[BE-NOTES/Python/Tecnologie/Context Manager|Context Manager]] per risorse con cleanup automatico.

## try / except / else / finally

```python
try:
    risultato = 10 / 0
except ZeroDivisionError:
    print("Non puoi dividere per zero")

# Catturare eccezioni specifiche
try:
    file = open("non_esiste.txt")
    contenuto = file.read()
except FileNotFoundError:
    print("File non trovato")
except PermissionError:
    print("Permesso negato")
except Exception as e:
    print(f"Errore generico: {e}")
else:
    print("Nessun errore!")  # esegue solo se NO eccezione
finally:
    print("Questo viene sempre eseguito")  # cleanup, chiusura risorse
```

## Gerarchia delle eccezioni

```
BaseException
├── SystemExit
├── KeyboardInterrupt
└── Exception
    ├── ArithmeticError (ZeroDivisionError, OverflowError)
    ├── LookupError (IndexError, KeyError)
    ├── ValueError
    ├── TypeError
    ├── OSError (FileNotFoundError, PermissionError)
    └── ...
```

## raise — lanciare eccezioni

```python
def dividi(a: int, b: int) -> float:
    if b == 0:
        raise ValueError("Il divisore non può essere 0")
    return a / b

# Creare eccezioni custom
class SaldoInsufficiente(Exception):
    """Sollevata quando il saldo non copre l'operazione."""
    def __init__(self, saldo: float, richiesto: float):
        self.saldo = saldo
        self.richiesto = richiesto
        super().__init__(f"Saldo {saldo} insufficiente per {richiesto}")

# Uso
try:
    raise SaldoInsufficiente(100, 200)
except SaldoInsufficiente as e:
    print(f"Mancano {e.richiesto - e.saldo}€")
```

## EAFP vs LBYL — spiegazione

Python predilige **E**asier to **A**sk **F**orgiveness than **P**ermission:

### LBYL (Look Before You Leap) — stile Java/C
Prima di fare l'operazione, controlli se può funzionare:
```python
def leggi_file_lbyl(path):
    if os.path.exists(path):          # controllo 1
        if os.access(path, os.R_OK):  # controllo 2
            with open(path) as f:
                return f.read()
        else:
            return "Permesso negato"
    else:
        return "File non trovato"
```
**Problema**: Tra il controllo e l'uso, lo stato potrebbe cambiare (race condition). In più, duplica la logica di gestione errori.

### EAFP (Easier to Ask for Forgiveness) — stile Python
Provi l'operazione, e se fallisce gestisci l'eccezione:
```python
def leggi_file_eafp(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        return "File non trovato"
    except PermissionError:
        return "Permesso negato"
```
**Vantaggi**: più conciso, niente race condition, lo stack trace mostra esattamente dove fallisce.

### Quando usare l'uno o l'altro
- **EAFP** per operazioni I/O (file, rete, DB) — le eccezioni sono comuni
- **LBYL** per validazione logica (controllare input utente) — se il fallimento è prevedibile e non eccezionale
- **EAFP è più Pythonico** — ma non è obbligatorio

## assert — spiegazione

`assert` verifica una condizione e lancia `AssertionError` se falsa:
```python
def set_eta(eta: int):
    assert eta >= 0, "L'età non può essere negativa"
    ...
```

### assert non è per validazione
```bash
python script.py       # assert funziona
python -O script.py    # assert DISABILITATO! (-O ottimizza via gli assert)
python -OO script.py   # anche i docstring vengono rimossi
```
Per validazione **reale** (produzione, sicurezza, dati utente), usa `if` + `raise`:
```python
def set_eta(eta: int):
    if eta < 0:
        raise ValueError("L'età non può essere negativa")
```

### Quando usare assert
- **Contratti interni**: "questa funzione presume che x sia positivo"
- **Invarianti**: condizioni che dovrebbero sempre essere vere
- **Test**: `pytest` usa ampiamente assert
- **Debug**: per catturare bug durante sviluppo, non per gestire input malevolo
