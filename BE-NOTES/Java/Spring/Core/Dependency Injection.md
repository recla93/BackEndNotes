# Dependency Injection

**Dependency Injection (DI)** = Spring **fornisce** le dipendenze a una classe invece di farsele creare da sola.

## Senza DI vs Con DI

```java
// ❌ SENZA DI: coupling stretto, difficile testare
public class TaskController {
    private TaskService service = new TaskService();  // creato manualmente
    // Non puoi cambiare implementazione
    // Non puoi mockare nei test
}

// ✅ CON DI: Spring fornisce la dipendenza
@RestController
public class TaskController {
    private final TaskService service;

    public TaskController(TaskService service) {  // Spring lo passa
        this.service = service;
    }
}
```

## Vantaggi

- **Basso accoppiamento** — la classe non sa come creare le dipendenze
- **Testabilità** — puoi passare mock nei test
- **Flessibilità** — cambi implementazione senza toccare il codice che usa l'interfaccia
- **Codice più pulito** — niente costruttori complessi o factory manuali

## 3 Tipi di Injection

### 1. Constructor Injection (✅ CONSIGLIATO)

```java
@Service
public class TaskService {
    private final TaskRepository repository;
    private final AuditService auditService;

    // Spring chiama questo costruttore con tutte le dipendenze
    public TaskService(TaskRepository repository, AuditService auditService) {
        this.repository = repository;
        this.auditService = auditService;
    }
}
```

**Vantaggi:** campi `final`, dipendenze obbligatorie, testabile senza Spring, impossible dimenticare una dipendenza.

### 2. Setter Injection

```java
@Service
public class TaskService {
    private TaskRepository repository;

    @Autowired
    public void setRepository(TaskRepository repository) {
        this.repository = repository;
    }
}
```

**Quando usarla:** dipendenze opzionali (es. logging, configurazione con default).

### 3. Field Injection (🚫 SCONSIGLIATO)

```java
@Service
public class TaskService {
    @Autowired
    private TaskRepository repository;  // reflection, campi non final
}
```

**Problemi:** impossibile testare senza Spring, campi non final, rompe incapsulamento, nasconde dipendenze.

## @Qualifier — più bean dello stesso tipo

```java
@Component
@Qualifier("csv")
public class TaskExporterCsv implements TaskExporter { ... }

@Component
@Qualifier("pdf")
public class TaskExporterPdf implements TaskExporter { ... }

@Service
public class TaskService {
    private final TaskExporter exporter;

    public TaskService(@Qualifier("csv") TaskExporter exporter) {
        this.exporter = exporter;
    }
}
```

## @Primary — bean di default

```java
@Component
@Primary  // usato se non specifico @Qualifier
public class TaskExporterCsv implements TaskExporter { ... }
```

## Risoluzione delle dipendenze

Spring risolve le dipendenze in 3 fasi:

```
1. Crea tutti i bean (costruttori)
2. Inietta le dipendenze (@Autowired, costruttori, setter)
3. Esegue @PostConstruct
```

Se manca una dipendenza → `NoSuchBeanDefinitionException` all'avvio.
Se ci sono dipendenze circolari (A→B→A) → `BeanCurrentlyInCreationException`.

## Problemi comuni

| Problema | Causa | Soluzione |
|---|---|---|
| **Circular dependency** | A→B→A | Refactoring, estrai interfaccia, usa `@Lazy` |
| **Bean ambiguo** | 2+ bean stesso tipo | `@Qualifier` o `@Primary` |
| **Bean mancante** | Dipendenza non trovata | Controlla `@ComponentScan` o `@Bean` |
| **Field injection** | Test impossibile senza Spring | Usa constructor injection |
| **@Autowired opzionale** | Dipendenza potrebbe non esistere | `@Autowired(required=false)` o `Optional<T>` |

