---
topic: "Stringhe — Java"
nav_prev: "[[Variabili e Tipi di Dati.md]]"
nav_next: "[[Operazioni ed Espressioni.md]]"
---

Le stringhe in Java sono oggetti **immutabili** della classe `String`. Ogni modifica crea una nuova stringa. Per operazioni frequenti su stringhe mutabili esistono `StringBuilder` e `StringBuffer`.

A differenza di C (`char*`), in Java le stringhe sono oggetti first-class con metodi built-in. A differenza di Python, l'uguaglianza con `==` confronta **riferimenti**, non contenuto.

## Creazione e confronto

```java
// Letterale (internato nello String Pool)
String s1 = "Ciao";

// Nuovo oggetto (NON internato)
String s2 = new String("Ciao");

// Confronto
s1 == s2               // false: riferimenti diversi
s1.equals(s2)          // true: stesso contenuto
s1.equalsIgnoreCase("CIAO")  // true
```

Usa sempre `.equals()` per confrontare contenuto, mai `==`. `==` confronta l'indirizzo di memoria. Il letterale `"..."` usa lo String Pool: stringhe identiche riusano lo stesso oggetto.

## Metodi principali

```java
String s = "  Java è potente!  ";

s.length()              // 18
s.charAt(0)             // 'J'
s.substring(0, 4)       // "Java" (stop escluso)
s.indexOf("Java")       // 2 (partenza da 0)
s.contains("potente")   // true
s.startsWith("Java")    // false (spazi iniziali)
s.trim()                // "Java è potente!"
s.toUpperCase()         // "  JAVA È POTENTE!  "
s.replace("potente", "fantastico")
s.split(" ")            // ["", "", "Java", "è", "potente!", "", ""]
```

`trim()` rimuove spazi iniziali/finali. `split()` divide su regex. I metodi restituiscono **sempre** nuove stringhe (immutabilità). `substring()` in Java 7+ crea una copia (prima condivideva il char[]).

## String pooling e interning

```java
String a = "hello";
String b = "hello";
String c = new String("hello");

a == b                  // true (stesso riferimento nello String Pool)
a == c                  // false (c è un nuovo oggetto)
a == c.intern()         // true (intern() recupera dallo String Pool)
```

Lo String Pool è un'ottimizzazione: le stringhe letterali vengono riutilizzate. `intern()` forza il recupero dallo String Pool. Utile per risparmiare memoria quando hai molte stringhe duplicate.

## StringBuilder e StringBuffer

```java
// ❌ LENTO: crea 3 oggetti intermedi
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // ogni iterazione crea un nuovo String!
}

// ✅ VELOCE: StringBuilder mutabile
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();

// StringBuffer = versione thread-safe (più lenta)
StringBuffer buffer = new StringBuffer();
```

In loop di concatenazione, `StringBuilder` è **decine di volte** più veloce di `+` o `concat()`. `StringBuffer` è sincronizzato (thread-safe) ma più lento. Usa `StringBuilder` salvo contesti multi-thread.

## String.format() e formatted

```java
// String.format() — stile C
String msg = String.format("Ciao %s, hai %d anni", "Alice", 30);

// text block (Java 15+)
String json = """
    {
        "nome": "Alice",
        "eta": 30
    }
    """;

// formatted() (Java 15+, metodo di istanza)
String msg2 = "Ciao %s, hai %d anni".formatted("Alice", 30);
```

`String.format()` è utile per template con parametri. I text block (`"""..."""`) preservano l'indentazione relativa. `formatted()` è syntactic sugar per `String.format()`.

## Errori comuni

- **`==` al posto di `.equals()`**: il confronto più comune. Java non ha operator overloading come C++.
- **Concat in loop con `+`**: crea N oggetti intermedi. Usa `StringBuilder`.
- **`String` come parametro mutabile**: `String` è immutabile. Se passi una stringa a un metodo, il metodo non può modificarla.
- **Dimenticare `trim()`**: input utente spesso arriva con spazi. Normalizza sempre.
- **`substring()` con indici invertiti**: `s.substring(5, 2)` lancia `StringIndexOutOfBoundsException`.
- **Confondere `length()` (metodo) con `length` (array)**: `String` usa `.length()`, array usano `.length`.

## Best Practices & Conventions

- Usa `.equals()` per confrontare stringhe, mai `==`.
- Usa `StringBuilder` per concatenazioni in loop o multi-step.
- Per confronti case-insensitive: `s1.equalsIgnoreCase(s2)`.
- Per input utente o dati esterni: chiama sempre `.trim()` prima di elaborare.
- Usa `String.format()` o text block per template complessi.
- Evita `new String("...")`: il costruttore esplicito è quasi sempre inutile.
- Per numeri, usa `String.valueOf(num)` invece di `"" + num`.
