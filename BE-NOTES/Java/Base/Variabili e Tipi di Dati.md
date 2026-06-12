# Variabili e Tipi di Dati

Una **variabile** è un contenitore con un nome che memorizza un valore di un tipo specifico. Java è **statically typed** — il tipo è dichiarato e non può cambiare.

## Dichiarazione

```java
// Dichiarazione e inizializzazione insieme
int x = 5;

// Dichiarazione separata
int x;
x = 5;

// Multipla dichiarazione (stesso tipo)
int a = 1, b = 2, c = 3;
```

## Tipi Primitivi (8)

Memorizzano il **valore direttamente** nello stack. Sono veloci e occupano poco spazio.

| Tipo | Dimensione | Valori | Esempio | Uso tipico |
|---|---|---|---|---|
| `byte` | 1 byte | -128 a 127 | `byte b = 100;` | Stream di byte, risparmio memoria |
| `short` | 2 byte | -32.768 a 32.767 | `short s = 1000;` | Raro, solo per compatibilità |
| `int` | 4 byte | -2¹⁵ a 2³¹-1 | `int i = 42;` | **Default per interi** |
| `long` | 8 byte | -2⁶³ a 2⁶³-1 | `long l = 100L;` | Numeri grandi, timestamp, ID |
| `float` | 4 byte | ±3.4E-38 a ±3.4E+38 | `float f = 3.14f;` | Raro (precisione limitata) |
| `double` | 8 byte | ±1.7E-308 a ±1.7E+308 | `double d = 3.14;` | **Default per decimali** |
| `char` | 2 byte | 0 a 65.535 (Unicode) | `char c = 'A';` | Un singolo carattere |
| `boolean` | ~1 bit | true/false | `boolean ok = true;` | Condizioni, flag |

**Regola:** per interi usa `int` (se serve di più → `long`). Per decimali usa `double` (se serve precisione → `BigDecimal`).

## Tipi Reference (Wrapper + Oggetti)

Memorizzano un **riferimento all'oggetto** nello heap. Sono più costosi ma offrono metodi e nullabilità.

```java
String nome = "Mario";                    // String è un reference type
ArrayList<Integer> lista = new ArrayList<>();
Persona p = new Persona();
```

Ogni primitivo ha un **wrapper class** corrispondente:

| Primitivo | Wrapper | Note |
|---|---|---|
| `int` | `Integer` | Autoboxing: `Integer x = 42` |
| `double` | `Double` | `Double x = 3.14` |
| `boolean` | `Boolean` | `Boolean x = true` |
| `long` | `Long` | `Long x = 100L` |
| ... | ... | Stessa logica per tutti |

### Autoboxing e Unboxing

```java
// Autoboxing: primitivo → wrapper automaticamente
Integer x = 42;  // Java fa: Integer.valueOf(42)

// Unboxing: wrapper → primitivo automaticamente
int y = x;       // Java fa: x.intValue()

// Attenzione: null in unboxing → NullPointerException!
Integer z = null;
int w = z;       // 💥 NullPointerException!
```

## Inizializzazione di default

Le variabili di **istanza** (campi di classe) hanno valori di default; le **locali** (dentro i metodi) NO!

```java
public class Persona {
    private int eta;         // 0 (default)
    private String nome;     // null (default)
    private boolean attivo;  // false (default)
}

public void metodo() {
    int x;                   // ❌ NON inizializzata!
    System.out.println(x);   // ERRORE in compilazione
}
```

## var — type inference (Java 10+)

```java
var nome = "Mario";              // String (inferito)
var lista = new ArrayList<String>(); // ArrayList<String>
var numeri = List.of(1, 2, 3);   // List<Integer>

// ❌ Non si può usare:
// var x;                        // senza inizializzazione
// var n = null;                 // tipo ambiguo
// var f = () -> "ciao";         // lambda ha bisogno di tipo target
```

## Casting e conversione

```java
// Widening (implicito, sicuro): int → long → double
int x = 100;
long y = x;          // OK: int è più piccolo di long

// Narrowing (esplicito, pericoloso): double → int
double d = 3.14;
int i = (int) d;     // i = 3 (perde i decimali!)

// Overflow silenzioso
int max = Integer.MAX_VALUE;  // 2.147.483.647
max = max + 1;                // -2.147.483.648 (overflow! nessun errore)
```

## Problemi comuni

| Problema | Esempio | Soluzione |
|---|---|---|
| **Overflow** | `int x = 2000000000 + 2000000000` | Usa `long` |
| **Precisione** | `0.1 + 0.2 != 0.3` | Usa `BigDecimal` per valute |
| **Null pointer** | `Integer x = null; x++` | Controlla null prima |
| **Comparazione** | `Integer x = 200; x == 200` | Usa `.equals()`, non `==` |
| **Divisione interi** | `5 / 2 = 2` | Usa `5.0 / 2` o `double` |