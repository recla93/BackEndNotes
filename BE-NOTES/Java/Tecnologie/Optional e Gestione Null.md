---
topic: "Optional e Gestione Null"
parent: "[[BE-NOTES/Java/Tecnologie/Core Concepts|Core Concepts]]"
nav_prev: "[[Java 21 - Pattern Matching.md]]"
nav_next: "[[Lombok - @Data e @Builder.md]]"
---


`Optional<T>` è un contenitore che rappresenta **esplicitamente l'assenza di un valore**. Introdotto in Java 8 per risolvere il problema dei `NullPointerException` silenziosi. Il suo scopo non è eliminare i null, ma **costringere il chiamante a gestire il caso "vuoto"**.

## Quando usarlo

`Optional` va usato **solo come tipo di ritorno** di metodi che possono non avere un risultato:

- **Repository JPA** — `Optional<Task> findById(Long id)` (l'oggetto potrebbe non esistere)
- **Service layer** — `Optional<User> findByEmail(String email)` (ricerca per campo opzionale)
- **Validazione** — `Optional<Error> validate(Task task)` (errore presente o assente)

**Quando NON usarlo:**
- **Campi di classe** — `Optional` non è serializzabile, causa problemi con JPA, Jackson, Lombok
- **Parametri di metodi** — meglio overloading o `@Nullable`
- **Collezioni** — una lista vuota (`Collections.emptyList()`) è più espressiva di `Optional<List<T>>`
- **Valori già noti** — se il valore è sempre presente, restituiscilo direttamente

## Creazione

```java
Optional<String> pieno = Optional.of("valore");          // lancia NPE se passi null
Optional<String> vuoto  = Optional.empty();               // contenitore vuoto
Optional<String> sicuro = Optional.ofNullable(valore);    // accetta null → Optional.empty()
```

Scegli `Optional.of()` se sei certo che il valore non sia null (fallisci subito se lo è). Scegli `Optional.ofNullable()` se il valore può venire da una fonte incerta.

## Consumo — le 3 vie

```java
// 1. Valore di default
Task task = repo.findById(id)
    .orElse(defaultTask);                       // sempre valutato

Task task = repo.findById(id)
    .orElseGet(() -> creaTaskDefault());         // valutato solo se vuoto

// 2. Eccezione se assente
Task task = repo.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Task", id));
    // pattern tipico nei service: se non trovi, lancia 404

// 3. Azione solo se presente
repo.findById(id)
    .ifPresent(task -> log.info("Trovato: {}", task));
```

## Trasformazioni sicure

`map()` e `flatMap()` permettono di **trasformare l'interno senza controllare il vuoto**:

```java
// Senza Optional: catena di null check
if (user != null && user.getTask() != null) {
    String title = user.getTask().getTitle();
}

// Con Optional: trasformazione sicura
String title = Optional.ofNullable(user)
    .map(User::getTask)      // se user è null → Optional.empty()
    .map(Task::getTitle)     // se task è null → Optional.empty()
    .orElse("Nessun task");
```

Se l'operazione restituisce un altro `Optional`, usa `flatMap()`:

```java
Optional<Task> task = userRepo.findByEmail(email)
    .flatMap(user -> taskRepo.findByUserId(user.getId()));
    // senza flatMap: Optional<Optional<Task>>
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `Optional.get()` senza controllo | `NoSuchElementException` a runtime | Chiami `.get()` su `Optional.empty()` | Usa `.orElse()`, `.orElseGet()`, o `.orElseThrow()` invece di `get()` diretto |
| `Optional` come campo di classe | `NotSerializableException` o errori JPA/Jackson | `Optional` non implementa `Serializable`; Lombok e JPA non lo gestiscono | Lascia il campo semplicemente `null` o con valore di default |
| `Optional` come parametro di metodo | Il chiamante ignora l'Optional e passa `null` lo stesso | `Optional` non impedisce `null` — il parametro può essere null | Usa overloading: `findByName(String)` e `findByName()` |
| `Optional` per collezioni | Ridondanza: `Optional<List<T>>` | Una lista vuota (`[]`) è già un contenitore che rappresenta assenza | Restituisci `Collections.emptyList()` invece di `Optional<List<T>>` |
| `orElse` con espressione costosa | Calcolo eseguito ANCHE quando l'Optional è pieno | `orElse(expensive())` valuta sempre l'argomento | Usa `orElseGet(() -> expensive())` — valutazione lazy solo se vuoto |
| `isPresent()`-`get()` invece di `ifPresent` | Codice verboso, più soggetto a errori | Pattern `if (opt.isPresent()) { opt.get()... }` | Preferisci `ifPresent()`, `map()`, `flatMap()`, o `orElseThrow()` |
| Usare `Optional.of()` con valore potenzialmente null | `NullPointerException` | `Optional.of(null)` lancia subito NPE | Usa `Optional.ofNullable(valore)` se il valore può essere null |

## In TaskMngr

- Ogni `findById()` del repository restituisce `Optional`
- I service chiamano `.orElseThrow()` per lanciare eccezioni gestite dal [[BE-NOTES/Java/Spring/Web/Global Exception Handler|Global Exception Handler]]
- Le trasformazioni `map()`/`flatMap()` sono preferite a `if (opt.isPresent()) { opt.get(); }`
- `Optional` non usato mai in entità JPA o DTO — campi semplicemente `null` o con valori di default
