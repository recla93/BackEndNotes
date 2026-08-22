---
topic: "Operatori ed Espressioni in C#"
nav_prev: "[[Tipi e Variabili.md]]"
nav_next: "[[Controllo di Flusso.md]]"
---

Gli operatori in C# sono simboli che producono un valore a partire da uno o più operandi. L'insieme è più ricco di Java: include operatori null-conditional, pattern matching, switch expression, e operatori di intervallo.

## Operatori aritmetici

## Perché esistono
Gli operatori aritmetici sono la base di ogni computazione numerica. C# li definisce per tutti i tipi numerici built-in e permette l'overload per tipi personalizzati.

| Operatore | Nome | Esempio | Risultato |
|:-:|---|---|---|
| `+` | Addizione | `5 + 3` | `8` |
| `-` | Sottrazione | `5 - 3` | `2` |
| `*` | Moltiplicazione | `5 * 3` | `15` |
| `/` | Divisione | `5 / 3` | `1` (intera!) |
| `%` | Modulo | `5 % 3` | `2` |
| `++` | Incremento | `i++` / `++i` | Post/pre |
| `--` | Decremento | `i--` / `--i` | Post/pre |

La divisione tra interi tronca il risultato: `5 / 3` dà `1`, non `1.666`. Per divisione reale, almeno un operando deve essere `double` o `float`:
```csharp
int a = 5, b = 3;
double r1 = a / b;       // 1.0 — divisione intera, poi cast implicito
double r2 = (double)a / b; // 1.666 — divisione reale
```

## Operatori di confronto e uguaglianza

```csharp
int x = 5, y = 10;
bool uguale = x == y;          // false
bool diverso = x != y;         // true
bool minore = x < y;           // true
bool maggiore = x > y;         // false
bool minoreUguale = x <= y;    // true
bool maggioreUguale = x >= y;  // false
```

Per i reference type, `==` verifica l'identità (stesso oggetto) a meno che non sia overloadato (come in `string`). Per value type confronta per valore. Usare `Equals()` per confronto logico, `ReferenceEquals()` per identità.

`==` su string è implementato come confronto per valore, ma solo perché la classe `string` ne fa l'overload. Su `object`, `==` confronta i riferimenti.

## Operatori logici (booleani)

```csharp
bool a = true, b = false;
bool e = a && b;       // AND: false (cortocircuito)
bool o = a || b;       // OR: true (cortocircuito)
bool not = !a;         // NOT: false
bool xor = a ^ b;      // XOR: true

// Senza cortocircuito (rari, usare solo per side effect)
bool and = a & b;      // valuta entrambi
bool or = a | b;       // valuta entrambi
```

`&&` e `||` usano cortocircuito: se il primo operando basta a determinare il risultato, il secondo non viene valutato. Usa `&` e `|` booleani solo quando il secondo operando ha un side effect necessario (raro, spesso codice puzzolente).

## Operatore condizionale ternario

```csharp
int eta = 20;
string stato = eta >= 18 ? "Maggiorenne" : "Minorenne";
// equivalente a:
// if (eta >= 18) stato = "Maggiorenne";
// else stato = "Minorenne";
```

Il ternario è un'espressione, non uno statement: deve restituire un valore. Non usarlo per side effect (es. `cond ? FaiA() : FaiB()`) — in quel caso usa un `if`.

## Operatori null-conditional e null-coalescing

Caratteristici di C#, gestiscono il null in modo conciso e sicuro:

```csharp
string? nome = OttieniNomeNullable();
int lunghezza = nome?.Length ?? 0;
// ?. = null-conditional: se nome è null, non chiama Length e restituisce null
// ?? = null-coalescing: se il risultato è null, usa 0 come default

// Esempio più complesso
var ordine = cliente?.Ordini?.FirstOrDefault(o => o.Data == oggi);
// Intera catena sicura — QUALSIASI anello null blocca e restituisce null
```

`?.` cortocircuita: se una qualsiasi parte della catena è null, restituisce null senza valutare il resto. Il risultato di `?.` è sempre un nullable.

## Switch expression (pattern matching)

Dalla sintassi `switch` tradizionale (statement) a quella **espressione** (C# 8+):

```csharp
// Tradizionale (statement)
string DescriviPunteggio(int voto)
{
    switch (voto)
    {
        case >= 90: return "Eccellente";
        case >= 70: return "Buono";
        case >= 50: return "Sufficiente";
        default: return "Insufficiente";
    }
}

// Switch expression (C# 8+)
string DescriviPunteggio(int voto) => voto switch
{
    >= 90 => "Eccellente",
    >= 70 => "Buono",
    >= 50 => "Sufficiente",
    _ => "Insufficiente"        // discard = default
};
```

La switch expression è **un'espressione**, produce un valore. Il `_` (discard) corrisponde al `default`. Il compilatore avverte se non tutti i casi sono coperti.

## Operatore di intervallo (range)

## Perché esiste
L'operatore `..` (C# 8+) semplifica l'estrazione di sub-intervalli da array, stringhe, span, e qualsiasi tipo che implementi l'indexing.

```csharp
int[] numeri = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
int[] primiTre = numeri[..3];       // { 0, 1, 2 }
int[] ultimiTre = numeri[^3..];     // { 7, 8, 9 } — ^3 = "terzultimo"
int[] centrali = numeri[3..7];      // { 3, 4, 5, 6 }

// Con stringhe
string testo = "Hello World";
string hello = testo[..5];          // "Hello"
string world = testo[^5..];         // "World"
```

`^` è l'operatore "index from end": `^1` è l'ultimo elemento, `^2` il penultimo, ecc.

## Operatori bitwise

```csharp
int a = 0b1100;     // 12
int b = 0b1010;     // 10
int and = a & b;    // 0b1000 = 8
int or  = a | b;    // 0b1110 = 14
int xor = a ^ b;    // 0b0110 = 6
int not = ~a;       // 0b...11110011 (complemento a 2)

int shiftSx = a << 2;    // 0b110000 = 48
int shiftDx = a >> 2;    // 0b0011 = 3
int shiftLog = a >>> 2;  // 0b0011 (unsigned, C# 11+)
```

## Precedenza operatori (dal più alto al più basso)

| Livello | Categoria | Operatori |
|:-:|---|---|
| 1 | Primari | `x.y`, `x?.y`, `f(x)`, `a[i]`, `x++`, `x--`, `new`, `typeof`, `nameof`, `sizeof`, `default` |
| 2 | Unari | `+x`, `-x`, `!x`, `~x`, `++x`, `--x`, `(T)x`, `^x`, `await` |
| 3 | Moltiplicativi | `*`, `/`, `%` |
| 4 | Additivi | `+`, `-` |
| 5 | Shift | `<<`, `>>`, `>>>` |
| 6 | Relazionali e type-testing | `<`, `>`, `<=`, `>=`, `is`, `as` |
| 7 | Uguaglianza | `==`, `!=` |
| 8 | AND logico | `&` |
| 9 | XOR logico | `^` |
| 10 | OR logico | `|` |
| 11 | AND condizionale | `&&` |
| 12 | OR condizionale | `||` |
| 13 | Coalescing | `??` |
| 14 | Ternario | `?:` |
| 15 | Assegnamento e lambda | `=`, `+=`, `-=`, `=>` |

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Divisione intera inaspettata** | `5 / 3 = 1` invece di `1.666` | Entrambi gli operandi sono interi | Castare almeno un operando a `double` |
| **NullReference con ternario** | Crash quando una branch è null | Il ternario non protegge dal null | Usare `?.` + `??` invece |
| **&& vs &** | Valutazione completa anche se non serve | Usare `&` o `|` invece di `&&`/`||` | Raro; preferire `&&`/`||` |
| **Precedenza errata** | `x + y * 2` valutato come `(x + y) * 2` | Confusione sulla priorità | Usare parentesi esplicite `()` |
| **^ come XOR vs potenza** | `2 ^ 3 = 1` invece di 8 | Confondere XOR con elevamento a potenza | `Math.Pow(2, 3)` o `double.Pow(2, 3)` |

## Best Practices

- **Usa parentesi** quando la precedenza non è ovvia — il codice si legge più di quanto si scrive
- **Preferisci switch expression** a switch statement quando possibile — più espressiva e type-safe
- **Usa `?.` e `??`** per catene sicure — eliminano pagine di `if (x != null)`
- **Non abusare del ternario** — va bene per assegnamenti semplici, ma se annidato diventa illeggibile
- **Per controlli di range**, usa lo switch con pattern `>=` e `<` — più leggibile di `if (x >= a && x < b)`
- **L'operatore `is`** per type testing è preferibile a `typeof` + cast: `if (x is Persona p) { p.Nome ... }`
