---
topic: "Strutture di Controllo — Python"
tags: [python, base, control-flow]
nav_prev: "[[Funzioni.md]]"
nav_next: "[[Liste Tuple Set Dict.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/controlflow.html](https://docs.python.org/3/tutorial/controlflow.html)

Python offre `if/elif/else`, `for`, `while`, `match` (3.10+). A differenza di Java/C#, **non usa parentesi** per le condizioni (usa `:` e indentazione) e **non ha `switch` tradizionale** (rimpiazzato da `match`).

Vedi anche: [[BE-NOTES/Python/Core Concepts/Funzioni|Funzioni]] per return e flow control nelle funzioni.

## if / elif / else

```python
eta = 18

if eta >= 18:
    print("Maggiorenne")
elif eta >= 13:
    print("Adolescente")
else:
    print("Bambino")

# Operatore ternario
status = "maggiorenne" if eta >= 18 else "minorenne"
```

## match (Python 3.10+)

Equivalente moderno dello switch-case:

```python
def rispondi(comando):
    match comando.lower():
        case "ciao":
            return "Ciao!"
        case "esci" | "exit":
            return "Arrivederci"
        case _:
            return "Non ho capito"

# Pattern matching strutturale
def analizza_punto(punto):
    match punto:
        case (0, 0):
            return "Origine"
        case (0, y):
            return f"Sull'asse Y a {y}"
        case (x, 0):
            return f"Sull'asse X a {x}"
        case (x, y):
            return f"Punto ({x}, {y})"
        case _:
            return "Non è un punto 2D"
```

## for loop

```python
# Iterare su lista
frutti = ["mela", "banana", "arancia"]
for frutto in frutti:
    print(f"Mangio {frutto}")

# Con indice (enumerate)
for i, frutto in enumerate(frutti):
    print(f"{i}: {frutto}")

# range()
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)
for i in range(0, 10, 2):   # 0, 2, 4, 6, 8
    print(i)

# Iterare su dict
persona = {"nome": "Mario", "eta": 25}
for key, value in persona.items():
    print(f"{key}: {value}")

# else nel for (opzionale, si esegue se NO break)
for n in range(10):
    if n == 5:
        break
else:
    print("Non ho trovato 5")  # NON eseguito (break ha interrotto)
```

## while loop

```python
contatore = 0
while contatore < 5:
    print(contatore)
    contatore += 1
else:
    print("Loop finito")  # opzionale, come for

# break / continue / pass
for i in range(10):
    if i == 5:
        break        # esce dal loop
    if i == 2:
        continue     # salta questa iterazione
    if i == 0:
        pass         # no-op, placeholder
    print(i)
```

## Best practice — spiegazioni

### `match` è strutturale: non solo switch-case
In Java/C, `switch` confronta solo valori. Il `match` di Python (3.10+) fa **pattern matching**: puoi destrutturare la forma dei dati:
```python
# Non solo valori
match comando:           # switch classico
    case "start": ...
    
# Ma anche strutture
match punto:             # destruttura tuple
    case (0, y): ...     # cattura y, controlla x==0

match utente:            # destruttura classi
    case {"ruolo": "admin", "nome": nome}: ...  # controlla chiave, cattura nome
```
È simile al pattern matching di Scala/Rust/Kotlin. Utile per: parser, state machine, routing, elaborazione dati complessi.

### Evita `else` dopo `for/while`
In Python, `for` e `while` possono avere un `else` che si esegue solo se il loop **non è stato interrotto da `break`**:
```python
for n in range(10):
    if n == 5:
        break
else:
    print("Non ho trovato 5")  # NON eseguito
```
A differenza di `if/else` (che è binario), il `for/else` non è intuitivo — anche sviluppatori esperti lo fraintendono. **Preferisci pattern espliciti**:
```python
trovato = False
for n in range(10):
    if n == 5:
        trovato = True
        break
if not trovato:
    print("Non ho trovato 5")
```

### `for i in range(len(lista))` è anti-pattern
In Java/C# scrivi `for (int i = 0; i < lista.length; i++)`. In Python, itera direttamente gli elementi:
```python
# Male — stile C inutile in Python
for i in range(len(frutti)):
    print(frutti[i])

# Bene — diretto
for frutto in frutti:
    print(frutto)

# Se serve l'indice:
for i, frutto in enumerate(frutti):
    print(i, frutto)
```
`enumerate()` restituisce coppie `(indice, elemento)`.

### `any()` e `all()` per controlli su sequenze
Invece di loop espliciti per controllare condizioni su tutti/gli elementi:
```python
numeri = [2, 4, 6, 8]

# Senza any/all
tutti_pari = True
for n in numeri:
    if n % 2 != 0:
        tutti_pari = False
        break

# Con any/all — più compatto e leggibile
tutti_pari = all(n % 2 == 0 for n in numeri)
qualche_pari = any(n % 2 == 0 for n in numeri)
```
`all()` restituisce `True` se tutti sono veri. `any()` se almeno uno è vero. Entrambi sono **lazy** (si fermano al primo risultato utile).

## Security

- **`eval()`** non è un'alternativa a `match` — mai eseguire codice da input
- **Attenzione a `input()` in loop** — Denial of Service: se un utente inserisce dati in un loop infinito, può consumare risorse. Usa timeout o limiti
- **`break` in loop annidati** rompe solo il loop più interno (usa `return` o flag per uscire da tutti)

## Cross-reference

- [[BE-NOTES/Python/Core Concepts/Funzioni|Funzioni]] — definizione, scope, lambda
- [[BE-NOTES/Python/Core Concepts/Errori e Eccezioni|Errori]] — try/except/finally
- [[BE-NOTES/Python/Core Concepts/Liste Tuple Set Dict|Collezioni]] — iterazione su dict/set
