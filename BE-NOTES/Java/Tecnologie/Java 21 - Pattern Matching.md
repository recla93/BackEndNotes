---
topic: "Pattern Matching — Java 21"
parent: "[[BE-NOTES/Java/Tecnologie/Core Concepts|Core Concepts]]"
nav_prev: "[[Java Records.md]]"
nav_next: "[[Optional e Gestione Null.md]]"
---


Il pattern matching permette di **verificare la forma di un oggetto ed estrarne i dati in un unico passo**. In Java 21 è stato esteso da `instanceof` (Java 16) allo `switch` come feature finale (JEP 441).

Prima del pattern matching, dovevi fare cast espliciti, controlli manuali e `null` check separati. Il pattern matching elimina tutto questo boilerplate.

## Quando usarlo

- **Dispatch condizionale sul tipo** — quando un metodo riceve un `Object` o un'interfaccia e deve comportarsi diversamente in base al tipo concreto
- **Decomposizione di oggetti composti** — estrarre campi da [[BE-NOTES/Java/Tecnologie/Core Concepts/Java Records|records]] senza chiamare getter
- **Sostituire catene `if-else instanceof`** — codice più leggibile e meno soggetto a errori
- **Coprire esplicitamente `null`** — Java 21 permette di gestire il caso null direttamente nello switch

**Quando NON usarlo:**
- Quando basta il polimorfismo classico (override di metodi)
- Su tipi semplici dove un casting diretto è più chiaro

## Type Patterns con instanceof

```java
// Prima (Java 7): cast manuale + controllo esplicito
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Java 16+: instanceof + pattern
if (obj instanceof String s) {
    System.out.println(s.length()); // s è già di tipo String
}
```

## Switch Pattern Matching (Java 21)

La vera potenza arriva con lo switch:

```java
public String describe(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "Intero negativo: " + i;
        case Integer i             -> "Intero positivo: " + i;
        case String s when s.isBlank() -> "Stringa vuota";
        case String s             -> "Stringa: " + s;
        case null                 -> "Valore nullo";
        default                   -> "Tipo sconosciuto: " + obj.getClass().getSimpleName();
    };
}
```

Le **guardie** (`when`) permettono di affinare il match: prima controlli il tipo, poi una condizione su quel tipo. L'ordine conta — Java valuta i casi in sequenza.

## Record Patterns

I record patterns decomprimono un [[BE-NOTES/Java/Tecnologie/Core Concepts/Java Records|record]] nei suoi componenti:

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}

// Estrazione annidata
if (obj instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
    double dx = x2 - x1;
    double dy = y2 - y1;
}
```

Un solo pattern match estrae le coordinate di entrambi i punti — senza chiamare `getStart()`, `getX()`, ecc.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Ordine sbagliato dei case | Un case generico "mangia" quelli specifici | Java valuta i case in sequenza; il primo match vince | Metti i case più specifici (con guardie) PRIMA di quelli generici |
| Guardia `when` sbagliata | Condizione logica errata: l'operatore sbagliato (es. `=` vs `==`) | `case Integer i when i = 0` è un assegnamento, non un confronto | Usa `==` nelle guardie, non `=` |
| Pattern matching non copre tutti i tipi possibili | Errore di compilazione "non exhaustive switch" | Lo switch deve essere esaustivo; manca un `default` o un branch per ogni tipo dell'unione | Aggiungi `default` o copri esplicitamente tutti i casi |
| Dimenticare `null` in switch | `NullPointerException` se il selettore è null | `null` non matcha nessun pattern, neanche `default` | Aggiungi `case null ->` esplicitamente (disponibile da Java 17+) |
| Usare `instanceof` pattern su tipo erased | Errore di compilazione: pattern type non valido | `if (obj instanceof List<String>)` non compila — i generics sono erasi a runtime | Pattern match su raw type: `instanceof List list` e poi caste con cautela |

## In TaskMngr

- [[BE-NOTES/Java/Spring/Web/ApiResponse Pattern|ApiResponse]] decomposto con pattern matching per gestire success/error
- Switch su enum `Status` con guardie per logiche condizionali
- Dispatch su eventi di dominio con record patterns
