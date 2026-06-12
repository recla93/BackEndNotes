# Service Layer

## Cos'è il Service Layer

Il **Service Layer** contiene la **logica di business**. Separa la logica da controller (troppo "di rete") e repository (troppo "di dati"):

```
Controller (HTTP)
    ↓ Request/Response
Service (Business Logic)
    ↓
Repository (Database)
```

## Esempio completo

```java
@Service
public class TaskService {
    private final TaskRepository repository;

    public TaskService(TaskRepository repository) {
        this.repository = repository;
    }

    // Logica di business: validazione
    public Task creaTask(CreateTaskRequest request) {
        if (request.getTitolo() == null || request.getTitolo().isBlank()) {
            throw new IllegalArgumentException("Titolo obbligatorio");
        }
        Task task = new Task();
        task.setTitolo(request.getTitolo());
        task.setStato(StatoTask.PENDING);
        return repository.save(task);
    }

    // Logica di business: regole applicative
    public Task completaTask(Long id) {
        Task task = repository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task " + id + " non trovato"));
        if (task.getStato() == StatoTask.COMPLETATO) {
            throw new IllegalStateException("Task già completato");
        }
        task.setStato(StatoTask.COMPLETATO);
        task.setDataCompletamento(LocalDateTime.now());
        return repository.save(task);
    }

    // Logica di business: transazioni
    @Transactional
    public void riassegnaTask(Long taskId, Long nuovoUtenteId) {
        Task task = repository.findById(taskId).orElseThrow();
        task.setAssegnatarioId(nuovoUtenteId);
        // Se un'eccezione viene lanciata DOPO il save, tutto viene rollbackato
        auditService.log("Task " + taskId + " riassegnato");
        repository.save(task);
    }
}
```

## Regole del Service Layer

| Regola | Spiegazione |
|---|---|
| **Niente request/response** | Il service non deve sapere di HTTP, `@RequestParam`, `HttpSession` |
| **Niente dipendenze web** | Nessun import da `javax.servlet` o `org.springframework.web` |
| **Niente dipendenze DB dirette** | Usa repository, non `EntityManager` o JDBC direttamente |
| **Ogni metodo = una operazione atomica** | Un metodo = un'azione business |
| **Transazioni qui** | `@Transactional` sul service, non sul controller |

## Vantaggi

- **Separazione delle responsabilità** — controller gestisce HTTP, service gestisce logica
- **Riutilizzabile** — lo stesso service può essere chiamato da un controller, un batch, o un comando CLI
- **Testabile** — testi la logica business senza partire il server HTTP
- **Transazioni centralizzate** — un solo punto dove gestire `@Transactional`
- **Eccezioni specifiche** — lancia `IllegalStateException`, `ResourceNotFoundException` invece di response HTTP

## Quando NON usare un Service

- **CRUD puri senza logica** — se il metodo fa solo `repository.save()`, potresti usare un controller + repository direttamente (ma la separazione resta comunque utile per consistenza)
- **Micro-ottimizzazione** — project piccoli con 1-2 entità: tenere service separato può sembrare overhead, ma è una buona abitudine

## Problemi comuni

| Problema | Sintomo | Soluzione |
|---|---|---|
| **Service troppo grande** | 500+ righe, 20+ metodi | Spezza in più service per dominio |
| **Service che chiama altro service** | Dipendenze cicliche | Riesamina il design, estrai interfacce |
| **Transazione troppo lunga** | Lenta, deadlock | Isola operazioni brevi, usa `@Transactional(propagation=REQUIRES_NEW)` |
| **Logica nel controller** | Controller con if/else e calcoli | Sposta nel service |
| **Service stateless** | Campi di istanza con stato | I service devono essere **stateless** (senza campi di istanza mutabili) |
