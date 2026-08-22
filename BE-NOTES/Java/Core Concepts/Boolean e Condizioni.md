---
topic: "Boolean e Condizioni"
nav_prev: "[[Operazioni ed Espressioni.md]]"
nav_next: "[[Costrutti di Iterazione.md]]"
---

Il tipo `boolean` può avere solo due valori: `true` o `false`. È la base di ogni decisione nel codice.

## Operatori di Confronto

Confrontano due valori e restituiscono un `boolean`:

| Operatore | Significato | Esempio vero | Esempio falso |
|---|---|---|---|
| `==` | Uguale a | `5 == 5` | `5 == 3` |
| `!=` | Diverso da | `5 != 3` | `5 != 5` |
| `<` | Minore di | `5 < 10` | `10 < 5` |
| `>` | Maggiore di | `10 > 5` | `5 > 10` |
| `<=` | Minore o uguale | `5 <= 5` | `6 <= 5` |
| `>=` | Maggiore o uguale | `5 >= 5` | `4 >= 5` |

**Attenzione:** per confrontare **oggetti** (String, Integer, DTO) usa `.equals()`, NON `==`:
```java
String a = new String("ciao");
String b = new String("ciao");
a == b         // false! == confronta i RIFERIMENTI (memoria), non il valore
a.equals(b)    // true — confronta il contenuto
```

## Operatori Logici

Combinano più condizioni booleane:

```java
// AND (&&) — TUTTE le condizioni devono essere true
if (eta > 18 && patente == true) {
    System.out.println("Puoi guidare");
}

// OR (||) — ALMENO UNA condizione deve essere true
if (giorno.equals("sabato") || giorno.equals("domenica")) {
    System.out.println("È weekend");
}

// NOT (!) — NEGA la condizione
if (!piove) {
    System.out.println("Usciamo");
}
```

### Short-circuit evaluation

`&&` e `||` sono **short-circuit**: se la prima condizione è sufficiente, la seconda NON viene valutata:

```java
// &&: se la prima è false, la seconda non viene valutata
if (nome != null && nome.length() > 5) { ... }
// Se nome è null, nome.length() non viene mai chiamato → salva NullPointerException!

// ||: se la prima è true, la seconda non viene valutata
if (isFileExists() || readFromCache()) { ... }
// Se isFileExists() è true, readFromCache() non viene chiamato
```

Usa short-circuit per proteggerti da NPE e ottimizzare le performance.

## If-Else

```java
if (condizione) {
    // Esecuzione se true
} else if (altraCondizione) {
    // Esecuzione se else if è true
} else {
    // Esecuzione se tutte le precedenti sono false
}

// Early return — meglio di if-else annidati
public String getStatusName(Task task) {
    if (task == null) return "N/A";
    if (task.isCompleted()) return "Completato";
    if (task.isInProgress()) return "In corso";
    return "Da fare";
}
```

**Switch Espressione (Java 14+) — meglio dello switch tradizionale:**

```java
// ✅ Switch expression — restituisce un valore, nessun break
String tipoGiorno = switch(giorno) {
    case "lunedì", "martedì", "mercoledì", "giovedì", "venerdì" -> "feriale";
    case "sabato", "domenica" -> "festivo";
    case null -> "giorno non valido";  // Java 17+: gestione null
    default -> "sconosciuto";
};

// ❌ Switch tradizionale — verboso, dimenticare break è un bug
switch(giorno) {
    case "lunedì":
        System.out.println("Inizio settimana");
        break;  // SENZA break, esegue anche il case successivo (fall-through)
    case "venerdì":
        System.out.println("Fine settimana");
        break;
    default:
        System.out.println("Giorno normale");
}
```

## Operatore Ternario

```java
// Se condizione è vera → valore1, altrimenti → valore2
String risultato = (eta >= 18) ? "Maggiorenne" : "Minorenne";

// Equivalente a:
String risultato;
if (eta >= 18) {
    risultato = "Maggiorenne";
} else {
    risultato = "Minorenne";
}
```

**Quando usarlo:** assegnazioni semplici con due alternative. **Quando NON usarlo:** logiche complesse o ternari annidati (illeggibili).

## Pattern Matching per instanceof (Java 16+)

```java
// ❌ Prima: cast esplicito
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// ✅ Ora: pattern matching — instanceof + dichiarazione in un passo
if (obj instanceof String s) {
    System.out.println(s.length());  // s è già String
}
```

## Problemi comuni

| Problema | Esempio | Soluzione |
|---|---|---|
| **Confronto stringhe con ==** | `s == "ciao"` | Usa `s.equals("ciao")` |
| **NullPointer in .equals()** | `s.equals("ciao")` se s è null | `"ciao".equals(s)` (sicuro) |
| **Dimenticare break in switch** | Fall-through inaspettato | Usa switch expression `->` |
| **Assegnamento = invece di ==** | `if (x = 5)` | Il compilatore avverte, ma fai attenzione |
| **Operatore logico sbagliato** | `if (x > 5 && x < 10)` funziona, ma `if (5 < x < 10)` NO | Java non supporta confronti a catena |