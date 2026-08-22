---
topic: "ApiResponse Pattern"
parent: "[[BE-NOTES/Java/Spring/Web/REST API Design|REST API Design]]"
nav_prev: "[[Validazione con Bean Validation.md]]"
nav_next: "[[Global Exception Handler.md]]"
---


Tutte le risposte API di [[TaskMngr]] seguono una **struttura uniforme**: un envelope che contiene l'esito dell'operazione, un messaggio, i dati e (opzionalmente) gli errori campo per campo.

## Quando usarlo

- **API REST pubbliche** — qualsiasi client (Angular, mobile, terze parti) deve poter fare parsing prevedibile
- **Errori di validazione** — comunicare esattamente quale campo è sbagliato e perché
- **Operazioni con esito variabile** — success/error con struttura identica
- **API versionata** — l'envelope permette di aggiungere campi senza rompere i client esistenti (es. `timestamp`, `requestId`)

**Quando NON usarlo:**
- API interne/microservizi (il throughput è più importante della struttura uniforme)
- Endpoint che restituiscono file binari (download, immagini)

## Struttura della risposta

```json
{
  "success": true,
  "message": "Operazione completata",
  "data": { "id": 1, "title": "Task esempio", "status": "TODO" },
  "errors": null
}
```

In caso di errore:
```json
{
  "success": false,
  "message": "Validazione fallita",
  "data": null,
  "errors": {
    "title": "Il titolo è obbligatorio",
    "email": "Formato email non valido"
  }
}
```

## Implementazione (Record Java)

```java
public record ApiResponse<T>(
    boolean success,            // true = operazione riuscita, false = errore
    String message,             // messaggio user-friendly
    T data,                     // payload (null se errore)
    Map<String, String> errors  // errori campo per campo (null se success)
) {
    // Factory methods per i casi più comuni
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Operazione completata", data, null);
    }

    public static <T> ApiResponse<T> success(T data, String message) {
        return new ApiResponse<>(true, message, data, null);
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null, null);
    }

    public static <T> ApiResponse<T> error(String message, Map<String, String> errors) {
        return new ApiResponse<>(false, message, null, errors);
    }
}
```

Il tipo generico `<T>` permette di usare `ApiResponse<TaskDto>`, `ApiResponse<Page<UserDto>>`, `ApiResponse<Void>` (quando non ci sono dati da restituire, es. DELETE).

## Uso nei controller

```java
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<TaskDto>> getTask(@PathVariable Long id) {
    TaskDto task = taskService.findById(id);
    return ResponseEntity.ok(ApiResponse.success(task));
}

@PostMapping
public ResponseEntity<ApiResponse<TaskDto>> createTask(
        @Valid @RequestBody TaskCreateRequest request) {
    TaskDto task = taskService.create(request);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(ApiResponse.success(task, "Task creato con successo"));
}

@DeleteMapping("/{id}")
public ResponseEntity<ApiResponse<Void>> deleteTask(@PathVariable Long id) {
    taskService.delete(id);
    return ResponseEntity.ok(ApiResponse.success(null, "Task eliminato"));
}
```

## Vantaggi rispetto a restituire raw data

| Aspetto | Raw data | ApiResponse |
|---|---|---|
| **Parsing lato client** | Ogni endpoint ha struttura diversa | Un unico parser in Angular |
| **Errori** | HTTP status + body variabile | Struttura fissa con `errors` |
| **Naming** | Convenzione diversa per ogni dev | Schema standardizzato |
| **Backward compat** | Cambiare struttura rompe i client | Aggiungi campi all'envelope |
| **Testing** | Assert diversi per ogni endpoint | Un unico `assertSuccess()` |

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| ApiResponse senza generics | Warning del compilatore, rischio cast errati | `ApiResponse` usato come raw type invece di `ApiResponse<T>` | Specifica il tipo: `ApiResponse<TaskDto>`, mai `ApiResponse` nudo |
| errors è una mappa ma il client lo tratta come stringa | Parsing JSON fallisce lato client | La struttura `errors` cambia da `null` a mappa | Il client deve gestire sia null che Map; documenta nel contratto API |
| Messaggio di errore che espone dettagli interni | Fuga di informazione in produzione | `e.getMessage()` passato direttamente all'utente | Messaggi user-friendly per il client, stack trace solo in log |
| Restituire data = null in caso di success | Il client fa check su null per ogni risposta | Il pattern prevede data null solo in errore | In success, restituisci sempre data (anche `Void`) |

## In TaskMngr

- Usato in OGNI controller REST
- Generics per tipo di dato specifico (`ApiResponse<TaskDto>`, `ApiResponse<Page<...>>`)
- Factory methods `success()` e `error()` nei service
- Il [[BE-NOTES/Java/Spring/Web/Global Exception Handler|Global Exception Handler]] restituisce `ApiResponse.error()` per tutte le eccezioni
