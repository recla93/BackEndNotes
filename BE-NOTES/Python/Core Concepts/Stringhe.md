---
topic: "Stringhe — Python"
tags: [python, base, stringhe, str]
nav_prev: "[[Variabili e Tipi.md]]"
nav_next: "[[Strutture di Controllo.md]]"
---
Riferimento ufficiale: [docs.python.org/3/library/stdtypes.html#text-sequence-type-str](https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str)

Le stringhe (`str`) sono sequenze **immutabili** di caratteri Unicode. A differenza di Java (`String` e `StringBuilder`) o C (`char*`), in Python le stringhe supportano nativamente Unicode, slicing, e un ricco set di metodi senza bisogno di classi helper.

In Python non esiste un tipo `char` separato: un singolo carattere è una stringa di lunghezza 1. La scelta tra `'virgolette singole'`, `"doppie"`, `'''triple'''` e `f"stringhe"` dipende dal contesto, non dal tipo.

Vedi anche:
[[BE-NOTES/Python/Core Concepts/Variabili e Tipi|Variabili e Tipi]],
[[BE-NOTES/Python/Tecnologie/Regex|Regex]],
[[BE-NOTES/Python/Funzionale/itertools|itertools]].

## Letterali e escaping

```python
# Singole o doppie sono equivalenti
s1 = 'Python'
s2 = "Python"

# Triple per stringhe multi-riga
s3 = """Riga 1
Riga 2
Riga 3"""

# Escape con backslash
s4 = 'l\'apostrofo'
s5 = "riga 1\nriga 2"

# Raw string ignora escaping (utile per regex/path)
raw = r"C:\Users\nome"
```

Le triple quote conservano gli a capo. Per stringhe lunghe senza newline usa la concatenazione implicita tra parentesi.

## Indicizzazione e slicing

```python
testo = "Python"

testo[0]    # 'P'
testo[-1]   # 'n' (ultimo)
testo[0:3]  # 'Pyt' (stop escluso)
testo[::-1] # 'nohtyP' (inverso)
testo[::2]  # 'Pto' (passo 2)
```

Lo slicing non solleva mai `IndexError`: indici fuori range vengono troncati silenziosamente. Un indice singolo fuori range invece solleva eccezione.

## Metodi principali

```python
s = "  Python è Potente!  "

s.lower()            # '  python è potente!  '
s.upper()            # '  PYTHON È POTENTE!  '
s.strip()            # 'Python è Potente!' (rimuove spazi laterali)
s.replace("Potente", "Fantastico")  # '  Python è Fantastico!  '
s.split()            # ['Python', 'è', 'Potente!']
s.startswith("Py")   # True
" ".join(["a", "b"]) # 'a b'
```

I metodi delle stringhe restituiscono **sempre una nuova stringa** — l'originale non viene mai modificata (immutabilità). `strip()` senza argomenti rimuove spazi, tab, newline. `split()` senza argomenti divide su qualsiasi whitespace e scarta vuoti.

## Concatenazione e interpolazione

```python
# Concatenazione diretta (sconsigliata in loop)
nome = "Alice" + " " + "Rossi"

# f-string (Python 3.6+, raccomandata)
eta = 30
print(f"Ciao, {nome}. Hai {eta} anni.")

# str.format()
print("Ciao, {}. Hai {} anni.".format(nome, eta))

# %-formatting (stile C, legacy)
print("Ciao, %s. Hai %d anni." % (nome, eta))
```

Le f-string sono più veloci e leggibili di `.format()` e `%`. Sono espressioni valutate a runtime, quindi accettano qualsiasi espressione Python dentro `{}`.

## Unicode e codifica

```python
# Python 3 gestisce nativamente Unicode
carattere = "ñ"
print(ord(carattere))   # 241 (codice Unicode)
print(chr(241))         # 'ñ'

# Codifica/decodifica
utf8_bytes = carattere.encode("utf-8")     # b'\xc3\xb1'
riconvertito = utf8_bytes.decode("utf-8")  # 'ñ'
```

`encode()` restituisce `bytes`, `decode()` torna a `str`. Usa sempre `utf-8` salvo rare eccezioni. `latin-1` (o cp1252) è comune in contesti legacy Windows.

## Errori comuni

- **`TypeError: can only concatenate str (not "int") to str`**: dimentichi di convertire numeri con `str()` prima della concatenazione. Usa le f-string per evitarlo.
- **`IndexError: string index out of range`**: accedi a un indice oltre `len(s) - 1`. Lo slicing invece gestisce il caso senza errore.
- **Modificare una stringa credendo sia mutabile**: `s[0] = 'P'` solleva `TypeError`. Crea una nuova stringa invece.
- **Confondere `None` con stringa vuota**: `if s:` è Falso sia per `None` che per `""` ma per ragioni diverse. Usa `if s is not None` quando `None` è un valore ammissibile.

## Best Practices & Conventions

- Usa sempre **f-string** per interpolazione (salvo template dinamici con `.format()`).
- Preferisci **singole** `'` per stringhe semplici, **doppie** `"` solo se la stringa contiene apostrofi.
- **Triple doppie** `"""` per docstring e multi-riga.
- Evita concatenazione con `+` in loop: accumula in una lista e usa `"".join(lista)`.
- Per path su Windows, usa raw string `r"C:\path"` o barre dritte `"C:/path"` (Python le accetta).
- Per controlli di vuoto: `if not s:` è idiomatico e copre sia `None` che stringa vuota.
