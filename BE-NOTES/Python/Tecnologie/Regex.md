---
topic: "Regex (Espressioni Regolari) — Python"
tags: [python, regex, re, pattern-matching]
---
Riferimento ufficiale: [docs.python.org/3/library/re.html](https://docs.python.org/3/library/re.html)

Il modulo `re` fornisce espressioni regolari in Python. Le regex sono pattern testuali che descrivono insiemi di stringhe: utili per validazione, estrazione, sostituzione e parsing veloce.

L'uso principale è in tre modalità: `re.match()` (all'inizio della stringa), `re.search()` (ovunque), `re.findall()` (tutte le occorrenze). Il Python usa la sintassi regex Perl-like (non POSIX).

Vedi anche:
[[BE-NOTES/Python/Core Concepts/Stringhe|Stringhe]],
[[BE-NOTES/Python/Tecnologie/Context Manager|Context Manager]].

## Pattern di base

```python
import re

# match: controlla solo all'inizio
re.match(r"Py", "Python")        # match -> oggetto Match
re.match(r"on", "Python")        # None (non all'inizio)

# search: cerca ovunque
re.search(r"on", "Python")       # match -> oggetto Match

# findall: tutte le occorrenze
re.findall(r"\d+", "a1 b2 c3")   # ['1', '2', '3']

# split: divide su pattern
re.split(r"\s+", "a  b   c")     # ['a', 'b', 'c']

# sub: sostituzione
re.sub(r"\d+", "NUM", "a1 b2")   # 'aNUM bNUM'
```

Usa sempre **raw string** `r"pattern"` per evitare doppio escaping (es. `\d` invece di `\\d`).

## Metacaratteri principali

```python
.          # Qualsiasi carattere (tranne \n)
^          # Inizio stringa
$          # Fine stringa
*          # 0 o più ripetizioni
+          # 1 o più ripetizioni
?          # 0 o 1 ripetizione (oppure lazy quantifier)
{n,m}      # Da n a m ripetizioni
\d         # Una cifra [0-9]
\w         # Lettera, cifra, underscore [a-zA-Z0-9_]
\s         # Whitespace (spazio, tab, newline)
\b         # Word boundary
|          # OR logico
[]         # Classe di caratteri
()         # Gruppo di cattura
(?:...)    # Gruppo non catturante
```

Esempio pratico:

```python
# Email semplice
pattern = r"([\w.]+)@([\w]+)\.(\w+)"
match = re.search(pattern, "Contatto: mario.rossi@email.com")
match.groups()    # ('mario.rossi', 'email', 'com')
match.group(0)    # 'mario.rossi@email.com' (intero match)
match.group(1)    # 'mario.rossi'
match.group(2)    # 'email'
```

## Pattern compilati (raccomandati)

```python
import re

# Compila una volta, usa N volte
email_re = re.compile(r"([\w.]+)@([\w]+)\.(\w+)")

# I metodi sono gli stessi, ma più veloci
matches = email_re.findall("a@b.com c@d.org")
# [('a', 'b', 'com'), ('c', 'd', 'org')]
```

Compilare il pattern con `re.compile()` è più efficiente se usi la stessa regex più volte. Inoltre i pattern compilati hanno flag pre-impostati.

## Flag comuni

```python
import re

re.IGNORECASE   # Ignora maiuscole/minuscole
re.MULTILINE    # ^ e $ su ogni riga
re.DOTALL       # . matcha anche \n
re.VERBOSE      # Pattern con spazi e commenti

# Combinabili con |
pattern = re.compile(r"""
    ^                  # Inizio stringa
    (\w+)              # Nome utente
    @                  # Chiocciola
    ([\w.]+)           # Dominio
    \.(\w+)            # TLD
    $
""", re.VERBOSE | re.IGNORECASE)
```

`re.VERBOSE` trasforma il pattern in multi-riga con commenti. Essenziale per regex complesse.

## Named group

```python
import re

pattern = re.compile(r"(?P<name>\w+)@(?P<domain>[\w.]+)\.(?P<tld>\w+)")
match = pattern.search("mario@email.com")

match.group("name")    # 'mario'
match.group("domain")  # 'email'
match.group("tld")     # 'com'
match.groupdict()      # {'name': 'mario', 'domain': 'email', 'tld': 'com'}
```

I named group rendono il codice auto-documentato e resilienti a cambi di ordine nei gruppi.

## Lookahead e lookbehind

```python
import re

# Positive lookahead: matcha solo se SEGUE il pattern
re.findall(r"\w+(?=@)", "mario@email.com")   # ['mario']

# Negative lookahead: matcha solo se NON segue
re.findall(r"\w+(?!@)", "mario@email.com")   # ['mari', 'email', 'com']

# Positive lookbehind: matcha solo se PRECEDE
re.findall(r"(?<=@)\w+", "mario@email.com")  # ['email']

# Negative lookbehind: matcha solo se NON precede
re.findall(r"(?<!@)\w+", "mario@email.com")  # ['mario', 'com']
```

I lookaround non consumano caratteri: sono asserzioni a larghezza zero. Utili per estrarre contesto senza includerlo.

## Backreference

```python
import re

# Nella stessa regex
re.search(r"(\w+) \1", "ciao ciao")    # match (stessa parola ripetuta)

# Nella sostituzione
re.sub(r"(\w+)@(\w+)", r"\1 at \2", "a@b.com")   # 'a at b.com'

# Con named group
re.sub(r"(?P<name>\w+)@", r"\g<name> at ", "a@b.com")  # 'a at b.com'
```

`\1`, `\2` etc. si riferiscono ai gruppi catturati. Nella stringa di sostituzione, usa `\1` o `\g<name>`.

## Errori comuni

- **Regex troppo complessa**: per validare email/URL usa una libreria dedicata. Le regex tendono all'esplosione combinatoria.
- **Dimenticare l'escape di caratteri speciali**: `.` matcha qualsiasi carattere, non il punto letterale. Usa `\.` per il punto.
- **`re.match()` vs `re.search()`**: `match` controlla solo l'inizio della stringa. Molti si aspettano che cerchi ovunque come `search`.
- **Non compilare pattern usati in loop**: `re.compile()` fuori dal loop evita ricompilazioni a ogni iterazione.
- **Greedy vs lazy**: `.*` è greedy (prende il più possibile). `.*?` è lazy (prende il minimo). Differenza critica in pattern con ripetizioni.

## Best Practices & Conventions

- Usa **raw string** `r"pattern"` per tutti i pattern regex.
- **Compila** pattern riutilizzati con `re.compile()`.
- Preferisci **named group** `(?P<name>...)` per pattern con 3+ gruppi.
- Usa `re.VERBOSE` per regex complesse con commenti.
- Non reinventare la ruota: per email, URL, date, usa librerie specializzate (`email-validator`, `python-dateutil`).
- Testa le regex su [regex101.com](https://regex101.com/) prima di scriverle in codice.
- Preferisci metodi di stringa (`str.startswith()`, `str.split()`) se fanno il lavoro — sono più leggibili.
