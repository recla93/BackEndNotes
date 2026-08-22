---
topic: "Programmazione Funzionale — Concetti Base"
tags: [python, functional, fp, pure-functions]
nav_next: "[[Lambda map filter reduce.md]]"
---
Riferimento ufficiale: [docs.python.org/3/howto/functional.html](https://docs.python.org/3/howto/functional.html)

Python non è puramente funzionale (non ha immutabilità forzata, tail-call optimization), ma supporta tutti i concetti chiave:
- Funzioni come oggetti di prima classe
- Higher-order functions (map, filter, reduce)
- Lambda expressions
- Closure e scope
- Immutabilità (per scelta, non imposizione)

La programmazione funzionale in Python è complementare a [[BE-NOTES/Python/OOP/Classi e Oggetti|OOP]]. La uso per: pipeline di dati, trasformazioni, logica predicativa, evitando side-effect e stato mutabile condiviso.

Vedi anche: [[BE-NOTES/Python/Funzionale/Lambda map filter reduce|Lambda]], [[BE-NOTES/Python/Funzionale/itertools|itertools]], [[BE-NOTES/Python/Funzionale/functools|functools]].

Python tratta le funzioni come oggetti di **prima classe**:

```python
def saluta(nome: str) -> str:
    return f"Ciao {nome}"

# Funzioni come valori
f = saluta
f("Mario")  # "Ciao Mario"

# Funzioni come argomenti
def applica(func, valore):
    return func(valore)

applica(saluta, "Anna")  # "Ciao Anna"

# Funzioni come ritorno
def crea_moltiplicatore(fattore: float):
    def moltiplica(valore: float) -> float:
        return valore * fattore
    return moltiplica

per_2 = crea_moltiplicatore(2)
per_2(5)  # 10
```

## Pure functions

Una funzione pura:
- restituisce sempre lo stesso output per lo stesso input
- non ha side effects (non modifica stato esterno)

```python
# Pura
def somma(a: int, b: int) -> int:
    return a + b

# Impura (side effect: stampa)
def somma_log(a: int, b: int) -> int:
    print(f"Sommo {a} e {b}")
    return a + b

# Impura (modifica input)
def aggiungi(lista: list[int], val: int) -> list[int]:
    lista.append(val)  # modifica l'argomento!
    return lista
```

## Immutabilità

```python
# Preferire copie a modifiche in-place
numeri = [1, 2, 3]

# Male:
numeri.append(4)  # modifica originale

# Meglio:
nuovi = [*numeri, 4]  # nuovo: [1,2,3,4]

# Con tuple (immutabili per natura)
coordinate = (10, 20)
# coordinate[0] = 5  # TypeError
```

## Composizione

Comporre funzioni piccole e pure per costruire logiche complesse:

```python
def maiuscola(s: str) -> str:
    return s.upper()

def punto_esclamativo(s: str) -> str:
    return s + "!"

def componi(*funcs):
    """Composizione di funzioni: f(g(h(x)))."""
    def applica(valore):
        for f in reversed(funcs):
            valore = f(valore)
        return valore
    return applica

grida = componi(maiuscola, punto_esclamativo)
grida("ciao")  # "CIAO!"
```
