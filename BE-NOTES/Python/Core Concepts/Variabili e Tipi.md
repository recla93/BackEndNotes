---
topic: "Variabili e Tipi — Python"
tags: [python, base, types, variables]
nav_next: "[[Funzioni.md]]"
---
Riferimento ufficiale: [docs.python.org/3/tutorial/introduction.html](https://docs.python.org/3/tutorial/introduction.html)

Python è a **tipizzazione dinamica** (il tipo è determinato a runtime) e **forte** (non converte automaticamente tra tipi incompatibili). Questo significa che una variabile può cambiare tipo durante l'esecuzione, ma operazioni come `"5" + 3` lanciano `TypeError`.

A differenza di Java/C#, non dichiari il tipo — lo assegni e basta. I [[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]] sono opzionali ma raccomandati per progetti seri.

## Tipi built-in

```python
# Numerici
eta = 25               # int (intero, senza limite di grandezza)
prezzo = 19.99         # float (virgola mobile, doppia precisione)
complesso = 3 + 4j     # complex (parte reale + immaginaria)

# Sequenze
nome = "Mario"         # str (stringa, immutabile)
numeri = [1, 2, 3]     # list (mutabile, ordinata)
coordinate = (10, 20)  # tuple (immutabile, ordinata)

# Mappatura
persona = {"nome": "Mario", "eta": 25}  # dict (key-value)

# Insiemi
colori = {"rosso", "blu", "verde"}  # set (non ordinato, valori unici)
colori_frozenset = frozenset([1, 2, 3])  # frozenset (immutabile)

# Booleani e None
attivo = True          # bool (True / False)
vuoto = None           # NoneType (assenza di valore)
```

## Sistema dinamico

Python è a **tipizzazione dinamica** e **forte**:

```python
valore = 5        # int
valore = "testo"  # stesso nome, ora str (dinamico)

# Ma non puoi mischiare tipi implicitamente:
5 + "3"  # TypeError: non puoi sommare int e str
```

## Operatori

```python
# Aritmetici
10 + 5   # 15
10 - 5   # 5
10 * 5   # 50
10 / 3   # 3.333... (float, anche se divisibili)
10 // 3  # 3 (divisione intera)
10 % 3   # 1 (modulo/resto)
2 ** 3   # 8 (esponente)

# Confronto
5 == 5   # True (uguale)
5 != 3   # True (diverso)
5 is 5   # True (identità in memoria)
5 is not 3  # True

# Logici
True and False  # False
True or False   # True
not True        # False

# Membership
5 in [1, 2, 3, 5]  # True
"a" in "ciao"      # True
```

## Stringhe

```python
# Dichiarazione
s1 = "doppi apici"
s2 = 'apici singoli'
s3 = """multi
riga"""

# Interpolazione (f-strings, Python 3.6+)
nome = "Mario"
print(f"Ciao {nome}")         # "Ciao Mario"
print(f"2 + 2 = {2 + 2}")     # "Ciao 4"
print(f"{nome:>10}")          # "     Mario" (padding destra)
print(f"{nome:.2}")           # "Ma" (prime 2 lettere)

# Metodi principali
testo = "Hello World"
testo.lower()            # "hello world"
testo.upper()            # "HELLO WORLD"
testo.replace("World", "Python")  # "Hello Python"
testo.split()            # ["Hello", "World"]
" ".join(["a", "b", "c"])  # "a b c"
testo.startswith("Hel")  # True
testo.strip()            # rimuove spazi ai lati

# Slicing
testo[0]      # "H"
testo[-1]     # "d" (ultimo carattere)
testo[0:5]    # "Hello" (da 0 a 4)
testo[::-1]   # "dlroW olleH" (rovescia)
```

## Type Hints (base)

```python
nome: str = "Mario"
numeri: list[int] = [1, 2, 3]
coords: tuple[float, float] = (41.9, 12.5)
attivo: bool = True
valore: int | None = None  # Union type (Python 3.10+)
```

## Best practice — spiegazioni

### F-strings: cosa sono e perché usarle
Prima di Python 3.6, per interpolare variabili in stringhe si usava `%` (stile C) o `.format()`:
```python
# % — vecchio stile, difficile da leggere
print("Ciao %s, hai %d anni" % (nome, eta))

# .format() — meglio ma verboso
print("Ciao {}, hai {} anni".format(nome, eta))

# f-strings (3.6+) — leggibile, veloce, potente
print(f"Ciao {nome}, hai {eta} anni")
```
Le f-strings sono **più veloci** (compilate a runtime), **più leggibili** (le variabili sono dentro la stringa), e supportano **espressioni**: `f"2+2 = {2+2}"`, formattazione: `f"{prezzo:.2f}"`, chiamate: `f"{nome.upper()}"`.

### `is None` vs `== None`: perché `is` è meglio
`is` controlla l'**identità in memoria** (è lo STESSO oggetto?), `==` controlla il **valore** (sono EQUIVALENTI?).
```python
x = None
x is None   # True — None è un singleton, unico oggetto in memoria
x == None   # True — ma potrebbe dare false positive se __eq__ è sovrascritto
```
`None` in Python è un **singleton** (esiste una sola istanza). `is` è più veloce e semanticamente corretto: stai chiedendo "è l'assenza di valore?", non "è uguale a qualcosa che vale None?".

### frozenset: quando serve
Un `set` normale è mutabile (puoi aggiungere/rimuovere elementi). Questo significa che **non può essere usato come chiave dict** (le chiavi devono essere immutabili). `frozenset` è la versione immutabile:
```python
set_mutabile = {1, 2, 3}
# dizionario = {set_mutabile: "valore"}  # TypeError: unhashable type: 'set'

set_immutabile = frozenset([1, 2, 3])
dizionario = {set_immutabile: "valore"}  # OK
```
Usalo quando hai bisogno di un insieme che funzioni da chiave in un dict o elemento in un altro set.

### Mutability matters: il problema dei default mutabili
Python valuta i parametri di default **una sola volta**, al momento della definizione della funzione (non ogni volta che la chiami):
```python
def sbagliato(elemento, lista=[]):  # [] creato UNA VOLTA
    lista.append(elemento)
    return lista

print(sbagliato(1))  # [1]
print(sbagliato(2))  # [1, 2] — OPS! La lista è la stessa dell chiamata precedente
```
Per evitarlo:
```python
def corretto(elemento, lista=None):
    if lista is None:
        lista = []
    lista.append(elemento)
    return lista
```
Lo stesso vale per `{}`, `set()`, e qualsiasi oggetto mutabile.

### Decimal vs Float: perché mai float per denaro
`float` in Python (e in tutti i linguaggi) è **binario** — alcuni numeri decimali non sono rappresentabili esattamente:
```python
0.1 + 0.2  # 0.30000000000000004 — arrotondamento!
```
Per denaro, calcoli finanziari, o qualsiasi cosa che richieda precisione esatta, usa `Decimal`:
```python
from decimal import Decimal
Decimal("0.10") + Decimal("0.20")  # Decimal('0.30') — esatto!
```
`Decimal` è più lento di `float`, ma preciso. `float` va bene per: misure fisiche, coordinate, percentuali approssimate.

### eval(): perché è pericoloso
`eval()` esegue **qualsiasi codice Python** passato come stringa:
```python
input_utente = "__import__('os').system('rm -rf /')"  # input malevolo
eval(input_utente)  # esegue il comando!
```
Non c'è modo di "sanificare" abbastanza l'input per `eval()` — è una vulnerabilità di **code injection**. Alternativa sicura: `int("42")`, `json.loads(dati)`, `ast.literal_eval(stringa)`.

### pickle: non deserializzare dati non fidati
`pickle` è il formato di serializzazione nativo di Python. Il problema: durante la deserializzazione, può eseguire codice arbitrario:
```python
import pickle
pickle.loads(dati_malevoli)  # può eseguire qualsiasi codice!
```
Regole:
- Usa `pickle` solo tra processi che si fidano al 100%
- Per dati da/verso l'esterno (API, file, DB), usa **JSON** o msgpack
- Alternativa sicura per dati Python complessi: `json` + encoding custom

### assert: perché non è sicurezza
`assert` è uno strumento di **debug**, non di validazione:
```python
def preleva(saldo, importo):
    assert importo > 0, "L'importo deve essere positivo"  # si disabilita con -O!
    return saldo - importo
```
Se avvii con `python -O script.py`, tutti gli `assert` vengono rimossi. Per validazione reale, usa `if` + `raise ValueError()`.

## Security

## Cross-reference

- [[BE-NOTES/Python/Core Concepts/Funzioni|Funzioni]] — scope LEGB, parametri, type hints
- [[BE-NOTES/Python/Core Concepts/Liste Tuple Set Dict|Liste Tuple Set Dict]] — approfondimento collezioni
- [[BE-NOTES/Python/Tecnologie/Type Hints|Type Hints]] — tipizzazione avanzata
- [[BE-NOTES/Python/Core Concepts/Errori e Eccezioni|Errori]] — gestione eccezioni
