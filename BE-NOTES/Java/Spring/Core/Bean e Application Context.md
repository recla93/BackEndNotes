---
topic: "Bean e Application Context"
nav_prev: "[[Libreria vs Framework.md]]"
nav_next: "[[Dependency Injection.md]]"
---

## Cos'è un Bean?

Un **Bean** è un oggetto **gestito da Spring** (creato, configurato, assemblato, e gestito nel ciclo di vita). Di default è **Singleton** — una sola istanza per tutta l'applicazione.

```
┌──────────────────────────────────────┐
│    Application Context (IoC Container)│
│                                      │
│  ┌────────────────────────────────┐  │
│  │ Bean: TaskService (singleton)  │  │
│  │ Bean: TaskRepository           │  │
│  │ Bean: TaskController           │  │
│  │ Bean: AuditService             │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

## Application Context

L'**Application Context** è il cuore di Spring: un registro (IoC Container) che contiene tutti i bean e gestisce:
- **Creazione** dei bean all'avvio (o su richiesta)
- **Iniezione** delle dipendenze tra bean
- **Ciclo di vita** (init, destroy)
- **Configurazione** (properties, profiles)

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(Application.class, args);

        // Puoi ottenere bean dal contesto
        TaskService service = ctx.getBean(TaskService.class);
    }
}
```

## Creazione di Bean — 3 modi

### 1. Annotazioni (component scanning)

```java
@Component          // Generico
@Service            // Service Layer
@Repository         // Data Access Layer
@Controller         // Spring MVC
@RestController     // REST API
```

Spring scansiona i package e crea bean per tutte le classi con queste annotazioni.

### 2. @Bean in @Configuration

```java
@Configuration
public class AppConfig {
    @Bean
    public TaskService taskService(TaskRepository repo) {
        // Controllo totale sulla creazione
        return new TaskService(repo);
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### 3. Factory method

```java
@Bean
public DataSource dataSource() {
    return DataSourceBuilder.create()
        .url("jdbc:mysql://localhost:3306/mydb")
        .username("root")
        .password("secret")
        .build();
}
```

## Scope dei Bean

| Scope | Descrizione | Creazione | Esempio |
|---|---|---|---|
| `singleton` (default) | Una istanza per tutta l'app | All'avvio | Service, Repository |
| `prototype` | Nuova istanza ogni richiesta | Su richiesta | Oggetti con stato |
| `request` | Una per richiesta HTTP | Per request | DTO di contesto |
| `session` | Una per sessione HTTP | Per sessione | Carrello utente |
| `application` | Una per ServletContext | Per app | Cache globale |

```java
@Service
@Scope("prototype")  // nuova istanza ogni @Autowired o ctx.getBean()
public class TaskProcessor {
    private String contesto;  // può avere stato
}
```

## Ciclo di vita di un Bean

```
1. Istanziazione (costruttore)
2. Iniezione dipendenze (@Autowired, setter, constructor)
3. @PostConstruct (init) — dopo che tutte le dipendenze sono state iniettate
4. Bean pronto all'uso
5. @PreDestroy (cleanup) — quando il contesto viene chiuso
```

```java
@Component
public class TaskService {
    public TaskService() {
        // 1. Costruttore
    }

    @Autowired
    public void setRepository(TaskRepository repo) {
        // 2. Iniezione dipendenze
    }

    @PostConstruct
    public void init() {
        // 3. Inizializzazione (es. caricare dati di configurazione)
    }

    @PreDestroy
    public void cleanup() {
        // 5. Pulizia (es. chiudere connessioni)
    }
}
```

## Problemi comuni

| Problema | Causa | Soluzione |
|---|---|---|
| **Circular dependency** | Bean A dipende da B, B dipende da A | Refactoring: estrai un terzo bean, o usa `@Lazy` su una delle dipendenze |
| **Scope mismatch** | Session bean iniettato in singleton bean | Usa `@Scope(proxyMode = ScopedProxyMode.TARGET_CLASS)` |
| **Bean non trovato** | Spring non scansiona il package dove si trova | Controlla `@SpringBootApplication` o `@ComponentScan` |
| **Bean iniettato prima di init** | Campo statico o inizializzazione in costruttore | Usa `@PostConstruct` invece del costruttore per logica di init |
| **Troppi bean dello stesso tipo** | `@Qualifier` mancante | Usa `@Primary` o `@Qualifier("nomeBean")` |
| **Prototype in singleton** | Singleton inietta prototype, ma prototype non viene ricreato | Usa `ObjectFactory<T>` o `Provider<T>` |
