---
topic: "Match Statement — Python 3.10+"
tags: [python, base, pattern-matching, match-case]
nav_prev: "[[Strutture di Controllo.md]]"
nav_next: "[[Liste Tuple Set Dict.md]]"
---
Riferimento ufficiale: [docs.python.org/3/reference/compound_stmts.html#match](https://docs.python.org/3/reference/compound_stmts.html#match)

Introdotto in Python 3.10, il `match` statement (pattern matching strutturale) permette di confrontare un valore contro una serie di **pattern** con sintassi dichiarativa. Non è un semplice `switch`-`case`: supporta decompilazione di strutture dati, binding di variabili, guardie e pattern annidati.

A differenza di `switch` in C/Java, `match` non ha fall-through implicito: ogni `case` è autonomo. Il primo pattern che matcha viene eseguito, senza bisogno di `break`.

Vedi anche:
[[BE-NOTES/Python/Core Concepts/Strutture di Controllo|Strutture di Controllo]],
[[BE-NOTES/Python/OOP/Classi e Oggetti|Classi e Oggetti]],
[[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]].

## Pattern semplici (costanti e literal)

```python
codice = 200

match codice:
    case 200:
        print("OK")
    case 404:
        print("Non trovato")
    case 500:
        print("Errore server")
    case _:
        print("Codice sconosciuto")
```

`_` è il pattern wildcard che matcha sempre (equivalente di `default`). L'ordine dei `case` conta: il primo match vincente viene eseguito. I literal matchano con `==`.

## Pattern con binding e tipo

```python
def analizza(valore):
    match valore:
        case 0:
            print("Zero")
        case int(n):
            print(f"Intero: {n}")
        case str(s):
            print(f"Stringa: {s}")
        case list(elementi):
            print(f"Lista con {len(elementi)} elementi")
        case _:
            print(f"Tipo sconosciuto: {type(valore).__name__}")
```

Il binding `int(n)` matcha solo se `valore` è un `int` **e** lega il valore a `n`. Se non ti serve il valore, usa `int()` senza variabile.

## Pattern con sequenze (list, tuple)

```python
punto = (10, 20)

match punto:
    case (0, 0):
        print("Origine")
    case (x, 0):
        print(f"Sull'asse X: {x}")
    case (0, y):
        print(f"Sull'asse Y: {y}")
    case (x, y):
        print(f"Punto: ({x}, {y})")
    case _:
        print("Non è un punto 2D")
```

I pattern sequenza decompongono tuple e liste in base alla struttura. `(x, 0)` matcha solo se il secondo elemento è 0 e lega il primo a `x`.

## Pattern con mapping (dict)

```python
utente = {"nome": "Alice", "ruolo": "admin"}

match utente:
    case {"ruolo": "admin"}:
        print("Accesso completo")
    case {"nome": nome, "ruolo": "editor"}:
        print(f"{nome} può modificare")
    case {"ruolo": role}:
        print(f"Ruolo sconosciuto: {role}")
    case _:
        print("Utente non valido")
```

Il pattern matcha anche se il dict contiene **più** chiavi di quelle specificate (match parziale). Per match esatto usa guardia: `case d if set(d.keys()) == {"nome", "ruolo"}:`.

## Guardie (`if` aggiuntivo)

```python
match punteggio:
    case int(n) if n >= 90:
        print("Eccellente")
    case int(n) if n >= 70:
        print("Buono")
    case int(n) if n >= 50:
        print("Sufficiente")
    case int(n):
        print("Insufficiente")
    case _:
        print("Punteggio non valido")
```

La guardia è un'espressione booleana dopo il pattern, preceduta da `if`. Il pattern matcha solo se anche la guardia è vera.

## Pattern su classi

```python
from dataclasses import dataclass

@dataclass
class Punto:
    x: float
    y: float

p = Punto(3.0, 4.0)

match p:
    case Punto(x=0, y=0):
        print("Origine")
    case Punto(x, y):
        print(f"Punto ({x}, {y})")
```

Il match per classi funziona per posizione (se `__match_args__` è definito) o per keyword. `dataclass` genera automaticamente `__match_args__`.

## Pattern nidificati

```python
dati = {
    "tipo": "rettangolo",
    "dimensioni": {"base": 5, "altezza": 3},
}

match dati:
    case {"tipo": "rettangolo",
          "dimensioni": {"base": b, "altezza": h}}:
        print(f"Area: {b * h}")
    case {"tipo": "cerchio",
          "dimensioni": {"raggio": r}}:
        print(f"Area: {3.14 * r ** 2}")
```

I pattern si annidano naturalmente: ogni sotto-struttura può avere il suo pattern. Utile per router, parser, e dispatch su API response.

## Errori comuni

- **Dimenticare `_` come default**: senza un `case _:`, il match non fa nulla se nessun pattern matcha. Nessun errore, ma comportamento silenzioso.
- **Confondere `|` (pipe) con OR logico**: `case 401 | 403 | 404:` matcha più valori. Non è un operatore bitwise sul valore.
- **Pattern troppo specifici**: `case {"nome": n, "eta": e}:` fallisce se manca una chiave. Usa `.get()` se le chiavi sono opzionali.
- **Usare variabili in pattern base**: `case n:` matcha sempre e lega a `n` (è un pattern, non un confronto). Se vuoi confrontare con una variabile, usa una guardia: `case _ if n == valore_esterno:`.

## Best Practices & Conventions

- Usa `match` per logiche con 3+ alternative e pattern strutturati. Per 2-3 opzioni semplici, `if`-`elif` è sufficiente.
- Preferisci pattern letterali sulle costanti (`case "admin":`) invece di `if ruolo == "admin":` quando la struttura è chiara.
- Sfrutta la decompilazione per estrarre dati da strutture annidate — evita catene di `.get()`.
- La guardia è l'unico modo per confrontare con variabili esterne al pattern.
- Per match su enum o costanti, usa pattern con dotted name: `case HttpStatus.OK:`.
