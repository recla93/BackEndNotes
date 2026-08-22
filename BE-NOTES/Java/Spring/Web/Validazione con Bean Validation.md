---
topic: "Validazione con Bean Validation"
parent: "[[BE-NOTES/Java/Spring/Web/REST API Design|REST API Design]]"
nav_prev: "[[DTO e Mappers.md]]"
nav_next: "[[ApiResponse Pattern.md]]"
---


La validazione lato server è **obbligatoria** — il client può mandare dati invalidi (volontariamente o no). Bean Validation (Hibernate Validator) permette di dichiarare regole di validazione sui DTO tramite annotazioni, senza scrivere `if` manuali.

## Quando serve

- **Ogni input utente** — POST, PUT, PATCH devono validare i dati in ingresso
- **Protezione da input malformati** — SQL injection (parziale), XSS, overflow
- **Feedback chiaro all'utente** — dire esattamente quale campo è sbagliato e perché
- **Prima linea di difesa** — prima che i dati arrivino al service e al database

La validazione lato **controller** è il primo filtro. La validazione lato **service** (logica di business) è il secondo. Non saltare mai nessuno dei due.

## Annotazioni standard

```java
public record TaskCreateRequest(
    @NotBlank(message = "Il titolo è obbligatorio")
    @Size(min = 3, max = 255, message = "Il titolo deve essere tra 3 e 255 caratteri")
    String title,

    @NotNull(message = "Lo status è obbligatorio")
    Status status,

    @Email(message = "Email non valida")
    String assigneeEmail,

    @PositiveOrZero(message = "La priorità deve essere positiva o zero")
    Integer priority
) { }
```

| Annotazione | Controlla | Esempio errore |
|---|---|---|
| `@NotBlank` | Stringa non null e non vuota | "Il titolo è obbligatorio" |
| `@NotNull` | Oggetto non null | "Lo status è obbligatorio" |
| `@Size(min, max)` | Lunghezza stringa/collezione | "Tra 3 e 255 caratteri" |
| `@Email` | Formato email valido | "Email non valida" |
| `@Positive`, `@PositiveOrZero` | Numeri positivi | "Deve essere positivo" |
| `@Past`, `@Future` | Date passate/future | "La data deve essere futura" |
| `@Pattern(regexp)` | Pattern regex | "Formato non valido" |

## Uso nei controller

```java
@PostMapping
public ResponseEntity<ApiResponse<TaskDto>> create(
        @Valid @RequestBody TaskCreateRequest request) {
    // Se request non supera la validazione, Spring NON chiama questo metodo
    // — lancia direttamente MethodArgumentNotValidException
    // che viene catturata dal GlobalExceptionHandler
    TaskDto task = taskService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED)
        .body(ApiResponse.success(task));
}
```

`@Valid` attiva la validazione. Se fallisce, il controller non viene eseguito — la risposta è automaticamente 400 con la lista degli errori.

## Validazione annidata

```java
public record TaskCreateRequest(
    @Valid    // ← valida ricorsivamente i campi di AssigneeInfo
    AssigneeInfo assignee
) { }

public record AssigneeInfo(
    @NotBlank String name,
    @Email String email
) { }
```

## Custom Validator

Per regole non coperte dalle annotazioni standard:

```java
@Target(FIELD)
@Retention(RUNTIME)
@Constraint(validatedBy = StatusSubsetValidator.class)
public @interface StatusSubset {
    String message() default "Status non valido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    Status[] anyOf();
}

public class StatusSubsetValidator
        implements ConstraintValidator<StatusSubset, Status> {

    private Status[] allowed;

    @Override
    public void initialize(StatusSubset constraint) {
        this.allowed = constraint.anyOf();
    }

    @Override
    public boolean isValid(Status value, ConstraintValidatorContext context) {
        if (value == null) return true;  // @NotNull gestisce il null
        return Arrays.asList(allowed).contains(value);
    }
}

// Uso
public record TaskFilterRequest(
    @StatusSubset(anyOf = {Status.TODO, Status.IN_PROGRESS})
    Status status
) { }
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `@Valid` dimenticato | I dati arrivano al service senza validazione | Spring non attiva Bean Validation senza `@Valid` | Aggiungi `@Valid` su ogni `@RequestBody` di POST/PUT/PATCH |
| Messaggio di errore non personalizzato | `must not be blank` invece di "Il titolo è obbligatorio" | Messaggio di default di Hibernate Validator | Specifica `message = "..."` in italiano su ogni annotazione |
| Validazione solo lato controller | Regola di business bypassabile via test diretto del service | "Tanto valida il controller" | Aggiungi validazione anche nel service per regole di dominio non banali |
| Custom validator che rifiuta null | Doppia segnalazione di errore (not null + custom) | Se `@NotNull` e custom validator gestisci null come invalido | Il custom validator deve restituire `true` se value == null; lascia a `@NotNull` la gestione |
| `@Size` su campo numerico | L'annotazione non funziona (è per stringhe/collezioni) | `@Size` non si applica a `Integer`, `int`, `Long` | Usa `@Min/@Max` per numeri, `@Size` per stringhe e collezioni |
| Validazione annidata senza `@Valid` | I campi del sotto-oggetto non vengono validati | Spring non valida ricorsivamente senza `@Valid` esplicito | Aggiungi `@Valid` sul campo che contiene l'oggetto annidato |

## In TaskMngr

- Tutti i request DTO hanno annotazioni di validazione
- `@Valid` su ogni controller POST/PUT/PATCH
- Errori catturati dal [[BE-NOTES/Java/Spring/Web/Global Exception Handler|Global Exception Handler]] e restituiti come `ApiResponse.error(message, errors)`
- `@Valid` annidato per oggetti composti
- Custom validators per logiche specifiche di dominio (es. status validi per transizione)
