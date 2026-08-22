---
topic: "Operazioni ed Espressioni"
nav_prev: "[[Variabili e Tipi di Dati.md]]"
nav_next: "[[Boolean e Condizioni.md]]"
---

Un'espressione è una combinazione di **valori, variabili, operatori e metodi** che produce un singolo valore.

## Operatori Aritmetici

Gli operatori aritmetici (`+`, `-`, `*`, `/`, `%`) lavorano su tipi numerici. **Attenzione alla divisione tra interi**: il risultato tronca la parte decimale.

```java
int somma = 10 + 5;        // 15
int diff = 10 - 5;         // 5
int prod = 10 * 5;         // 50
int div = 10 / 3;          // 3 (divisione INTERA, tronca i decimali!)
double divEx = 10.0 / 3;   // 3.333... (se almeno un operando è double)
int resto = 10 % 3;        // 1 (modulo: resto della divisione)
```

## Operatori di Assegnamento

Gli operatori composti (`+=`, `-=`, `*=`, `/=`, `%=`) combinano assegnamento e operazione: `x += 3` equivale a `x = x + 3`.

```java
int x = 5;       // Assegnamento base
x += 3;          // x = x + 3 → 8
x -= 2;          // x = x - 2 → 6
x *= 4;          // x = x * 4 → 24
x /= 3;          // x = x / 3 → 8
x %= 5;          // x = x % 5 → 3
```

## Operatori di Incremento/Decremento

`++` incrementa di 1, `--` decrementa di 1. La differenza tra pre- e post- è: **pre-incremento** (`++x`) prima modifica poi usa il valore; **post-incremento** (`x++`) prima usa il valore poi modifica.

```java
int x = 5;
x++;             // Post-incremento: usa x poi incrementa → x = 6
++x;             // Pre-incremento: incrementa poi usa → x = 7
x--;             // Post-decremento
--x;             // Pre-decremento

// Differenza in espressioni:
int a = 5;
int b = a++;     // b = 5, a = 6 (prima assegna, poi incrementa)
int c = ++a;     // c = 7, a = 7 (prima incrementa, poi assegna)
```

**Consiglio:** usa `++` e `--` solo come istruzioni standalone, non dentro espressioni complesse.

## Concatenazione di String

`+` su stringhe fa **concatenazione**, non somma. Se un operando è stringa, Java converte l'altro in stringa. La valutazione è **da sinistra a destra**: `10 + 5 + "ciao"` somma prima i numeri (15) poi concatena "15ciao".

```java
System.out.println(10 + 5 + "ciao");     // 15ciao (prima somma, poi concatena)
System.out.println("ciao" + 10 + 5);     // ciao105 (tutto stringa da sinistra)
System.out.println("ciao" + (10 + 5));   // ciao15 (parentesi hanno priorità)
```

## Precedenza degli Operatori

Dall'alta alla bassa priorità:

| Priorità | Operatori | Associatività |
|---|---|---|
| 1 (alta) | `()`, `[]`, `.` | sinistra |
| 2 | `++`, `--`, `+` (unario), `-` (unario), `!` | destra |
| 3 | `*`, `/`, `%` | sinistra |
| 4 | `+`, `-` (binario) | sinistra |
| 5 | `<`, `>`, `<=`, `>=` | sinistra |
| 6 | `==`, `!=` | sinistra |
| 7 | `&&` | sinistra |
| 8 | `\|\|` | sinistra |
| 9 | `=`, `+=`, `-=` ... | destra |

```java
int x = 5 + 3 * 2;      // 11 (non 16 — * ha priorità su +)
int y = (5 + 3) * 2;    // 16 (usa le parentesi per chiarire)
```

**Regola:** quando non sei sicuro della precedenza, usa le **parentesi**.

## Type Promotion nelle espressioni

Java promuove automaticamente i tipi per evitare perdita di precisione:

```java
byte a = 10;
byte b = 20;
// a e b vengono promossi a int prima della somma!
byte c = (byte) (a + b);  // serve cast esplicito

// Promozione: il tipo più "grande" vince
int + long → long
int + double → double
float + double → double
int + float → float
```

## Problemi comuni

| Problema | Esempio | Spiegazione |
|---|---|---|
| **Divisione intera** | `5 / 2 = 2` | Usa `5.0 / 2` |
| **Modulo con negativi** | `-5 % 2 = -1` | Il segno segue il dividendo in Java |
| **Overflow silenzioso** | `2_000_000_000 + 2_000_000_000 = -294967296` | Nessuna eccezione, usa `long` o `Math.addExact()` |
| **Precedenza errata** | `x < 10 && y > 5` | Le parentesi risolvono: `(x < 10) && (y > 5)` |
| **Associatività** | `x = y = z = 5` funziona perché `=` è destra-associativo | `x = (y = (z = 5))` |