---
topic: "Eccezioni — Java"
nav_prev: "[[Operazioni ed Espressioni.md]]"
nav_next: "[[Generics.md]]"
---

Le eccezioni in Java sono eventi che interrompono il flusso normale del programma. A differenza di C (codici di errore ignorabili), Java forza la gestione delle eccezioni **checked** a compile-time tramite `try-catch-finally` o dichiarazione `throws`.

Esistono tre categorie: **checked** (verificate in compilazione), **unchecked** (`RuntimeException`, non verificate), e **errori** (`Error`, problemi JVM). Un metodo deve dichiarare `throws` per le checked che non gestisce internamente.

## Gerarchia

```
Throwable
  ├─ Error (irrecuperabili: OutOfMemoryError, StackOverflowError)
  └─ Exception
       ├─ RuntimeException (unchecked: NullPointerException, IllegalArgumentException)
       └─ altre (checked: IOException, SQLException)
```

`Error` non va mai catturato (sono problemi JVM irrecuperabili). `RuntimeException` e sottoclassi sono unchecked (non obbligatorio gestirle). Tutte le altre `Exception` sono checked (obbligatorio gestirle o dichiararle).

## Try-catch-finally

```java
// Base
try {
    int risultato = 10 / 0;
} catch (ArithmeticException e) {
    System.err.println("Errore: " + e.getMessage());
} finally {
    System.out.println("Eseguito sempre");
}

// Multi-catch (Java 7+)
try {
    // codice che può lanciare IOException o SQLException
} catch (IOException | SQLException e) {
    System.err.println("Errore I/O o DB: " + e.getMessage());
}
```

`finally` viene eseguito sempre (anche con `return` nel `try`). Il multi-catch riduce duplicazione. Cattura eccezioni specifiche prima di generiche: l'ordine dei `catch` conta.

## Try-with-resources (Java 7+)

```java
// ❌ OLD: finally per chiudere risorse
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    System.out.println(reader.readLine());
} finally {
    if (reader != null) reader.close();
}

// ✅ NEW: try-with-resources (AutoCloseable)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    System.out.println(reader.readLine());
}
// reader viene chiuso automaticamente
```

Try-with-resources chiude automaticamente le risorse che implementano `AutoCloseable` (tutte le I/O, JDBC, ecc.). Più compatto, sicuro anche in caso di eccezione.

## Checked vs Unchecked

```java
// Checked: obbligatorio gestire
public void leggiFile() throws IOException {  // ok
    Files.readString(Path.of("file.txt"));
}

// Unchecked: non obbligatorio (ma può essere catturata)
public void dividi(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("b non può essere 0");
}
```

Le checked rappresentano condizioni esterne prevedibili (file non trovato, rete assente). Le unchecked rappresentano errori di programmazione (parametri nulli, indici fuori range).

## Custom exception

```java
// Checked
public class SaldoInsufficienteException extends Exception {
    public SaldoInsufficienteException(String message) {
        super(message);
    }
    public SaldoInsufficienteException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Unchecked
public class UtenteNonTrovatoException extends RuntimeException {
    public UtenteNonTrovatoException(Long id) {
        super("Utente non trovato: " + id);
    }
}
```

Usa checked exception per condizioni che il chiamante dovrebbe gestire. Usa unchecked per errori che il chiamante non può prevenire a compile-time. Includi sempre la causa originale (`cause`) per non perdere lo stack trace.

## Best practice: fail-fast e null-check

```java
public void processa(String input) {
    Objects.requireNonNull(input, "input non può essere null");

    // Uso di Optional per assenza di valore
    Optional<String> risultato = trova(input);
    String valore = risultato.orElseThrow(
        () -> new UtenteNonTrovatoException(input)
    );
}
```

`Objects.requireNonNull()` è preferibile a `if (x == null) throw ...`. `Optional` evita `null` come valore di ritorno ambiguo. Previeni le eccezioni dove possibile (fail-fast) invece di catturarle.

## Suppressed exceptions (try-with-resources)

```java
try (AutoCloseable a = new A(); AutoCloseable b = new B()) {
    // Se sia try che close() lanciano eccezioni,
    // l'eccezione di close() è "soppressa" e accessibile con getSuppressed()
}
```

Quando sia il blocco `try` che il `close()` lanciano eccezioni, l'eccezione di `close()` viene soppressa. Puoi accedervi con `e.getSuppressed()`. Le eccezioni soppresse sono un dettaglio implementativo importante per debugging.

## Errori comuni

- **Catturare `Exception` o `Throwable` generico**: nasconde bug. Cattura sempre l'eccezione più specifica.
- **Inghiottire l'eccezione**: `catch (Exception e) {}` senza log. Logga almeno con `logger.error()`.
- **`return` in `finally`**: sovrascrive qualsiasi eccezione o return dal `try`. Non farlo mai.
- **Checked troppo generiche dichiarate con `throws Exception`**: costringe il chiamante a catturare genericamente. Sii specifico.
- **Lanciare `new Exception()` invece di una sottoclasse**: le API pubbliche devono lanciare eccezioni significative.
- **Usare eccezioni per controllo di flusso**: `try { ... } catch (ExpectedException) { ... }` è un anti-pattern. Usa controlli condizionali.

## Best Practices & Conventions

- Cattura eccezioni **specifiche** non generiche.
- Logga sempre l'eccezione completa (`logger.error("msg", e)`), non solo `e.getMessage()`.
- Usa **try-with-resources** per tutte le risorse che implementano `AutoCloseable`.
- Preferisci **unchecked** per errori di programmazione, **checked** per condizioni esterne recuperabili.
- Non usare eccezioni per controllo di flusso.
- Documenta le eccezioni lanciate con `@throws` nei JavaDoc.
- Mantieni eccezioni a strati: cattura nel service, rilancia con contesto aggiuntivo (exception chaining).
