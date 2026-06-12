---
topic: "Global Exception Handler"
parent: "[[BE-NOTES/Java/Spring/Web/REST API Design|REST API Design]]"
---

# Global Exception Handler

In un'applicazione Spring, le eccezioni possono venire da ovunque: service, repository, validazione, sicurezza. Senza un handler centralizzato, ogni controller dovrebbe gestirle con `try-catch` — codice ripetitivo e fragile. `@RestControllerAdvice` risolve il problema intercettando TUTTE le eccezioni non gestite e convertendole in una risposta standard.

## Quando serve

- **Sempre** — ogni API REST dovrebbe avere un global exception handler
- Sostituisce `try-catch` sparsi in 20 controller con un unico punto di gestione
- Garantisce che ogni errore restituisca una [[BE-NOTES/Java/Spring/Web/ApiResponse Pattern|ApiResponse]] coerente

## Implementazione

```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody (restituisce JSON)
public class GlobalExceptionHandler {

    // 404 — risorsa non trovata
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiResponse<Void> handleNotFound(ResourceNotFoundException ex) {
        return ApiResponse.error(ex.getMessage());
    }

    // 400 — validazione fallita (cattura errori da @Valid)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        // errors = {"title": "Il titolo è obbligatorio", "email": "Formato email non valido"}
        return ApiResponse.error("Validazione fallita", errors);
    }

    // 403 — accesso negato
    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ApiResponse<Void> handleForbidden(AccessDeniedException ex) {
        return ApiResponse.error("Accesso negato");
    }

    // 401 — autenticazione fallita
    @ExceptionHandler(BadCredentialsException.class)
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public ApiResponse<Void> handleUnauthorized(BadCredentialsException ex) {
        return ApiResponse.error("Credenziali non valide");
    }

    // 409 — conflitto (es. OptimisticLockException)
    @ExceptionHandler(OptimisticLockException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ApiResponse<Void> handleConflict(OptimisticLockException ex) {
        return ApiResponse.error("La risorsa è stata modificata da un altro utente. Ricarica e riprova.");
    }

    // 500 — errore generico (catch-all)
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResponse<Void> handleGeneral(Exception ex) {
        log.error("Errore imprevisto: {}", ex.getMessage(), ex);  // log completo su server
        return ApiResponse.error("Errore interno del server");    // mai esporre dettagli al client
    }
}
```

## Eccezioni custom

Per separare i tipi di errore, creiamo eccezioni specifiche per dominio:

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, Long id) {
        super(resource + " non trovato con id: " + id);
    }
}

@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BusinessRuleException extends RuntimeException {
    public BusinessRuleException(String message) {
        super(message);
    }
}
```

Nei service:
```java
public TaskDto getTask(Long id) {
    return taskRepository.findById(id)
        .map(mapper::toDto)
        .orElseThrow(() -> new ResourceNotFoundException("Task", id));
    // Se task non esiste → 404 con messaggio chiaro
}
```

## Cosa NON fare

```java
// ❌ catch generico che mostra dettagli interni
return ApiResponse.error("SQL error: " + ex.getSQLState());

// ❌ try-catch in ogni controller
@GetMapping
public ResponseEntity<?> get(...) {
    try { return service.findById(id); }
    catch (Exception e) { return errorResponse(e); }
}

// ❌ stack trace nell'HTTP response
return ApiResponse.error(ex.toString());  // espone dettagli implementativi
```

## In TaskMngr

- `ResourceNotFoundException` → 404 (task, utente, team non trovati)
- `BadCredentialsException` → 401 (credenziali errate)
- `AccessDeniedException` → 403 (permessi insufficienti)
- `MethodArgumentNotValidException` → 400 con errori campo per campo
- `OptimisticLockException` → 409 (concorrenza)
- `Exception` generica → 500 (con log interno)
- Messaggi user-friendly in italiano
