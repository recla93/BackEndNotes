---
topic: "Spring Boot Best Practices — sicurezza, scalabilità, leggerezza"
parent: "[[BE-NOTES/Java/Spring/Boot/Spring Boot|Spring Boot]]"
nav_prev: "[[Dipendenze con Maven.md]]"
nav_next: "[[TaskMngr Approfondimenti.md]]"
---


Regole e convenzioni per scrivere applicativi Spring Boot **sicuri, scalabili e leggeri**. Ogni regola include il motivo ("perché") e quando eventualmente derogare.

---

## 1. Architettura e Organizzazione

### 1.1 Package by feature, non by layer

```
❌ Cattivo (by layer):
  controller/UserController.java
  service/UserService.java
  repository/UserRepository.java
  model/User.java

✅ Buono (by feature):
  user/
    User.java
    UserController.java
    UserService.java
    UserRepository.java
    dto/UserRequest.java
    dto/UserResponse.java
```

**Perché:** la coesione è più alta. Quando modifichi una feature, modifichi file vicini. In un package by layer, una modifica a User tocca 4 package diversi. In più, `user/` può essere estratto in un modulo separato se cresce.

**Quando derogare:** in progetti molto piccoli (< 5 entità) la differenza è trascurabile.

### 1.2 Separa API interna da API pubblica

```java
// /api/internal/** — solo tra microservizi (auth interna)
// /api/public/** — accessibile da terze parti
// /api/v1/** — versione stabile
```

Usa un prefisso unico in `server.servlet.context-path` o `spring.web.request-matcher-prefix`.

### 1.3 Entry point pulito

```java
@SpringBootApplication
public class TaskMngrApplication {

    public static void main(String[] args) {
        SpringApplication.run(TaskMngrApplication.class, args);
    }
}
```

Nessuna logica nell'entry point. Niente `@Enable*` a caso — Spring Boot auto-configura già.

---

## 2. Configurazione

### 2.1 Externalizza tutto

```properties
app.jwt.secret=${JWT_SECRET}
app.jwt.expiration-ms=${JWT_EXPIRATION_MS:86400000}
spring.datasource.url=${DATABASE_URL:jdbc:postgresql://localhost:5432/taskmngr}
```

**Perché:** la stessa immagine Docker gira in dev, staging e prod cambiando solo env var. Hardcodare significa rebuildare per ogni ambiente.

### 2.2 @ConfigurationProperties > @Value

```java
// ❌ @Value sparso ovunque
@Value("${app.jwt.secret}")
private String jwtSecret;

// ✅ @ConfigurationProperties centralizzato
@ConfigurationProperties(prefix = "app.jwt")
public class JwtProperties {
    @NotBlank
    private String secret;
    @Positive
    private long expirationMs;

    // getter e setter obbligatori
}
```

**Perché:** `@ConfigurationProperties` valida al startup (fail-fast), raggruppa proprietà correlate, è testabile in isolamento. `@Value` sparge stringhe magiche e non valida.

### 2.3 Profili ambientali minimi

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    hikari:
      maximum-pool-size: ${DB_POOL_SIZE:10}

---
server:
  port: 8080
spring:
  devtools:
    restart:
      enabled: true

---
server:
  port: 8080
spring:
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate  # MAI update in prod!
```

Usa `application-{profile}.yml` ereditando dal base. Non copiare tutto in ogni profilo — solo le differenze.

---

## 3. Dependency Injection

### 3.1 Constructor injection sempre

```java
// ✅ Buono: immutabile, testabile, esplicito
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
}

// ❌ Cattivo: field injection (non testabile senza reflection)
@Autowired private UserRepository userRepository;
```

**Perché:** costruttore esplicito → dipendenze obbligatorie, immutabilità (`final`), testabile con `new UserService(mockRepo, mockEncoder)`. Field injection nasconde dipendenze e richiede reflection nei test.

**Quando derogare:** mai. Se hai troppe dipendenze (> 5-6), il costruttore grande è un sintomo — la classe fa troppo, non il costruttore.

### 3.2 Qualifier solo quando serve

Se hai due implementazioni della stessa interfaccia:

```java
@Primary  // default
@Service
public class DefaultTaskService implements TaskService { }

@Service
@Qualifier("special")
public class SpecialTaskService implements TaskService { }
```

Usa `@Primary` per il default (`@Autowired TaskService` prende quello) e `@Qualifier("special")` solo dove serve l'altro.

### 3.3 Niente circolari

Dipendenza circolare = errore di progettazione. Sintomi:
- `A → B → A`
- Costruttori che non possono essere risolti

**Fix:** estrai l'interfaccia comune, spezza in 3 classi, o ripensa la responsabilità.

---

## 4. REST API

### 4.1 Naming e HTTP Methods

| Operazione | HTTP | Endpoint | Corpo | Risposta |
|---|---|---|---|---|
| Create | POST | `/api/resources` | Request body | 201 Created + location header |
| Read (tutti) | GET | `/api/resources` | — | 200 OK + array |
| Read (uno) | GET | `/api/resources/{id}` | — | 200 OK |
| Update (full) | PUT | `/api/resources/{id}` | Request body | 200 OK |
| Update (parziale) | PATCH | `/api/resources/{id}` | Request body | 200 OK |
| Delete | DELETE | `/api/resources/{id}` | — | 204 No Content |

- **Plurale** per risorse collettive: `/api/users` non `/api/user`
- **Sostantivi** non verbi: `/api/orders` non `/api/getOrders`
- **CamelCase** per campi JSON (o snake_case se il frontend preferisce)
- **No version nell'URL** se puoi usare content negotiation (`Accept: application/vnd.company.v1+json`)

### 4.2 DTO sempre, entity mai in input/output

```java
// ❌ Mai esporre l'entità
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) { return userService.findById(id); }

// ✅ Usa DTO
@GetMapping("/{id}")
public UserResponse getUser(@PathVariable Long id) { return userService.findById(id); }
```

**Perché:** l'entità è un dettaglio di persistenza. Esporla crea coupling, rischia di esporre dati sensibili (password), e rende difficile cambiare lo schema DB senza rompere l'API.

**Regola:** `UserResponse` contiene solo campi che l'utente deve vedere. `UserRequest` contiene solo campi che l'utente può inviare. Mai lo stesso oggetto per input e output.

### 4.3 Validazione al confine

```java
public record CreateUserRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8, max = 128) String password,
    @NotBlank @Size(max = 100) String name
) {}

@PostMapping
public ResponseEntity<UserResponse> create(@RequestBody @Valid CreateUserRequest request) {
    // il corpo è già validato, nessun if all'interno
}
```

**Perché:** validare al confine (controller) garantisce che solo dati puliti entrino nel service. Validare dentro il service significa doppio lavoro e logica sparsa.

### 4.4 Paginazione consistente

```java
@GetMapping
public Page<UserResponse> list(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id,asc") String[] sort
) {
    var sortOrders = Sort.by(
        Arrays.stream(sort)
            .map(s -> s.contains(",")
                ? new Sort.Order(Sort.Direction.fromString(s.split(",")[1]), s.split(",")[0])
                : new Sort.Order(Sort.Direction.ASC, s))
            .toList()
    );
    Pageable pageable = PageRequest.of(page, Math.min(size, 100), sortOrders);
    return userService.findAll(pageable);
}
```

**Regole:**
- Default: page=0, size=20, max size=100 (proteggi da attacchi di paginazione enorme)
- Risposta: `Page<T>` include contenuti, totalElements, totalPages, page, size
- Ordine: param sort = `campo,direzione` (es. `?sort=createdAt,desc&sort=name,asc`)

---

## 5. Database e JPA

### 5.1 ddl-auto: update solo in dev

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update

spring:
  jpa:
    hibernate:
      ddl-auto: validate  # o usa Flyway
```

**Perché:** `update` in prod può cancellare dati per errore, non traccia le modifiche, e non gestisce rollback. Usa **Flyway** (o Liquibase) per migrazioni versionate in produzione.

### 5.2 Entity: POJO semplice, niente logica di business

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

    @Enumerated(STRING)
    @Column(nullable = false, length = 10)
    private Role role;

    @OneToMany(mappedBy = "owner", cascade = ALL, orphanRemoval = true)
    private List<Task> tasks = new ArrayList<>();

    protected User() {}  // obbligatorio per JPA

    public User(String email, String password, Role role) {
        this.email = email;
        this.password = password;
        this.role = role;
    }

    // getter — niente setter pubblici per campi immutabili
    public Long getId() { return id; }
    public String getEmail() { return email; }
    // ...
}
```

**Regole JPA:**
- Costruttore vuoto protected (JPA lo richiede)
- `@GeneratedValue(strategy = IDENTITY)` per PostgreSQL/MySQL (SEQUENCE per Oracle)
- Campi immutabili (id, createdAt) solo getter, niente setter
- `@Column(nullable = false)` per ogni campo obbligatorio
- `@Column(unique = true)` per vincoli di unicità espliciti
- `orphanRemoval = true` su `@OneToMany` (cancella figli orfani)

### 5.3 Fetch LAZY di default

```java
@ManyToOne(fetch = LAZY)      // default per ManyToOne (va bene)
@OneToMany(mappedBy = "x")     // default LAZY da JPA 2.0 (va bene)
@OneToOne(fetch = LAZY)       // default EAGER → cambia in LAZY
```

**Perché:** EAGER carica tutto sempre, anche quando non serve. Peggiora le performance, causa N+1, e rende impossibile ottimizzare query per caso d'uso.

### 5.4 Previeni N+1 con JOIN FETCH o EntityGraph

```java
// ❌ N+1: N query per i task di N utenti
List<User> users = userRepo.findAll();
for (User u : users) {
    u.getTasks().size();  // una query per ogni utente!
}

// ✅ JOIN FETCH: una query sola
@Query("SELECT u FROM User u JOIN FETCH u.tasks")
List<User> findAllWithTasks();
```

Oppure con `@EntityGraph`:

```java
@EntityGraph(attributePaths = "tasks")
@Query("SELECT u FROM User u")
List<User> findAllWithTasks();
```

### 5.5 Indici e vincoli

```sql
-- Gli indici composti vanno valutati per le query reali
CREATE INDEX idx_tasks_owner_status ON tasks(owner_id, status);

-- Unique constraint su business key
ALTER TABLE users ADD CONSTRAINT uk_users_email UNIQUE (email);
```

In JPA:

```java
@Table(name = "tasks", indexes = {
    @Index(name = "idx_tasks_owner_status", columnList = "owner_id,status")
})
```

### 5.6 Connection Pool (HikariCP)

HikariCP è il default di Spring Boot 2+. Configurazioni base:

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10        # dipende dal workload
      minimum-idle: 5
      idle-timeout: 300000         # 5 minuti
      connection-timeout: 30000    # 30 secondi timeout connessione
      max-lifetime: 1800000        # 30 minuti (consigliato < DB timeout)
      pool-name: TaskMngrPool
      leak-detection-threshold: 60000  # 60 secondi — logga warning se una connessione è trattenuta
```

**Regola d'oro:** `pool-size = T * (C - 1)`, dove T = numero di thread/istanze, C = connessioni medie per richiesta. In pratica, **10-15 per microservizio è quasi sempre giusto**. Più connessioni = più contesa sul DB, non più velocità.

---

## 6. Gestione Errori

### 6.1 @ControllerAdvice centralizzato

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(NOT_FOUND.value(), ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        var errors = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .toList();
        return new ErrorResponse(BAD_REQUEST.value(), "Validation error", errors);
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        log.error("Errore inatteso: {}", ex.getMessage(), ex);  // logga lo stack!
        return new ErrorResponse(INTERNAL_SERVER_ERROR.value(), "Errore interno");
    }
}
```

**Regole:**
- Non esporre stack trace o dettagli interni (information leak)
- Logga sempre lo stack dell'errore prima di restituire un 500
- Usa un formato consistente per ErrorResponse
- Converti eccezioni di basso livello (JPA, SQL) in eccezioni di dominio

### 6.2 Codici di errore vs messaggi

```java
// ✅ Errore strutturato
{
    "code": 404,
    "message": "User with id 42 not found",
    "timestamp": "2026-06-12T10:30:00Z",
    "errors": []  // errori di validazione specifici
}

// ❌ Messaggio generico
{ "error": "Not Found" }
```

---

## 7. Logging

### 7.1 Livelli di log

| Livello | Quando usare |
|---|---|
| `TRACE` | Debug dettagliato (solo in dev, mai in prod) |
| `DEBUG` | Diagnostica sviluppo |
| `INFO` | Eventi rilevanti: startup, shutdown, cambio di stato importante |
| `WARN` | Situazioni anomale ma recuperabili (es. prima di un fallback) |
| `ERROR` | Errore che richiede attenzione (es. eccezione inaspettata, chiamata esterna fallita) |

**Mai usare `log.error` per errori previsti** (es. "utente non trovato" = WARN o INFO).

### 7.2 Log strutturato (JSON)

Per produzione, configura Logback con encoder JSON:

```yaml
logging:
  pattern:
    console:  # in dev, testo normale
    file:     # in prod, JSON
```

**Perché:** log JSON è ricercabile e analizzabile da ELK, Grafana Loki, Datadog, Splunk. Log testo normale è leggibile ma non strutturato.

### 7.3 Mai loggare dati sensibili

```java
// ❌ MAI
log.info("Login: user={}, password={}", email, password);
log.info("JWT token: {}", token);

// ✅ OK
log.info("Login attempt: email={}", email);
log.info("JWT issued: userId={}, role={}", userId, role);
```

---

## 8. Caching

### 8.1 Spring Cache Abstraction

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        var caffeine = Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(1000)
            .recordStats();
        return new CaffeineCacheManager("users", "tasks", "references");
    }
}
```

```java
@Cacheable(value = "users", key = "#id")
public UserResponse findById(Long id) {
    // chiamata al DB solo se non in cache
}

@CacheEvict(value = "users", key = "#id")
public void delete(Long id) {
    // rimuove dalla cache su modifica
}

@CacheEvict(value = "users", allEntries = true)
public UserResponse update(UserRequest request) {
    // invalida tutta la cache users
}
```

### 8.2 Quando NON usare cache

- **Dati che cambiano frequentemente** — il costo di invalidazione supera il beneficio
- **Dati sensibili** — la cache può esporli a utenti non autorizzati (es. dati multi-tenant)
- **Operazioni di scrittura** — cache solo letture
- **Dati con permessi variabili** — se ogni utente vede dati diversi, la cache perde senso

---

## 9. Async e Scheduling

### 9.1 @Async per operazioni non bloccanti

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        var executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

```java
@Service
public class NotificationService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String body) {
        // non blocca il thread HTTP
        return CompletableFuture.completedFuture(null);
    }
}
```

**Quando usare:** email, notifiche push, generazione report, elaborazione file — tutto ciò che l'utente non deve aspettare.

**Quando NON usare:** operazioni che devono essere completate prima di rispondere all'utente. Preferisci thread pool dedicato (non quello di default).

### 9.2 @Scheduled

```java
@Component
public class CleanupTasks {

    private final TaskRepository taskRepository;

    @Scheduled(cron = "0 0 3 * * ?")  // ogni notte alle 3
    @Transactional
    public void deleteExpiredTasks() {
        taskRepository.deleteByExpiredBefore(LocalDateTime.now());
    }
}
```

**Regole:**
- Usa cron expression esplicite, non `fixedRate` per operazioni notturne
- Assicurati che il job sia idempotente (eseguirlo due volte non causa danni)
- Considera `@Scheduled(zone = "Europe/Rome")` per non dipendere dal fuso del server

---

## 10. Test

### 10.1 Piramide dei test

```
        ╱╲
       ╱ E2E ╲         ← pochi (percorsi critici)
      ╱────────╲
     ╱ Integration ╲    ← alcuni (DB, API esterna)
    ╱────────────────╲
   ╱   Unit Test      ╲  ← molti (logica di business)
  ╱──────────────────────╲
```

### 10.2 Unit test: Mock ciò che non è tuo

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldThrowWhenEmailAlreadyExists() {
        var request = new CreateUserRequest("existing@test.com", "password123", "Test");
        when(userRepository.findByEmail("existing@test.com"))
            .thenReturn(Optional.of(new User()));

        assertThrows(DuplicateEmailException.class, () -> userService.create(request));
    }
}
```

### 10.3 Integration test: @SpringBootTest sì, ma mirato

```java
@SpringBootTest
@AutoConfigureTestDatabase(replace = REPLACE_ANY)  // H2 in memoria
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindByEmail() {
        userRepository.save(new User("test@test.com", "pass", Role.USER));

        var found = userRepository.findByEmail("test@test.com");

        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("test@test.com");
    }
}
```

### 10.4 Testcontainer per DB reali (vs H2)

```java
@SpringBootTest
@Testcontainers
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("testdb");

    @DynamicPropertySource
    static void configure(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    // test con PostgreSQL reale, non H2
}
```

**Perché:** H2 non supporta tutte le feature di PostgreSQL (JSONB, array, full-text search). I test passano in H2 ma falliscono in prod. Testcontainer risolve il problema.

---

## 11. Errori Comuni e Anti-Pattern

| Anti-pattern | Problema | Soluzione |
|---|---|---|
| `@Autowired` su field | Dipendenze nascoste, non testabile | Constructor injection |
| `ddl-auto=update` in prod | Perdita dati, nessun versioning | Flyway o validate |
| Entity esposta in API | Coupling API-DB, leak dati sensibili | DTO |
| N+1 query | Prestazioni degradate | JOIN FETCH, EntityGraph |
| Transazioni giganti | Lock lunghi, deadlock | Transazioni brevi e mirate |
| `@PostMapping` per cancellazioni | Confonde il client, viola REST | `@DeleteMapping` |
| Ignorare eccezioni nel catch | Errori silenziosi, debugging impossibile | Logga sempre |
| `@Data` di Lombok su entity | `equals/hashCode` con ID null | `@Getter @Setter` manuali |
| `@Transactional` su troppi metodi | Lock e connessioni tenute per troppo tempo | Solo dove servono |
| `Thread.sleep()` in test | Test lenti, instabili, non deterministici | Awaitility, mock |
| Package by layer | Bassa coesione, alto accoppiamento | Package by feature |
| Pool connessioni > 30 | Contesa sul DB, nessun beneficio | 10-15 per istanza |
| `application.properties` con secret | Credenziali nel repository | Env var o vault |

---

## 12. Checklist Sicurezza

- [ ] Secret in env var o vault, mai nel codice
- [ ] Password hashate con BCrypt (o Argon2 se disponibile)
- [ ] CSRF disabilitato solo per API stateless (JWT). Abilitato per session-based.
- [ ] CORS configurato con origini specifiche, mai `Access-Control-Allow-Origin: *`
- [ ] Rate limiting sulle route di login/register
- [ ] Input validation sempre al confine (controller + DTO `@Valid`)
- [ ] SQL injection prevenuta da JPA/JDBC parameterized queries (mai concatenazione)
- [ ] XSS prevenuto: non restituire mai HTML, escape nei template
- [ ] HTTPS obbligatorio in produzione (redirect HTTP→HTTPS)
- [ ] JWT: secret lungo (>256 bit), exp breve, niente dati sensibili nel payload
- [ ] `SecurityFilterChain` con route pubbliche esplicite e `anyRequest().authenticated()`
- [ ] `@PreAuthorize` su endpoint sensibili (delete, modify altri utenti)
- [ ] `@ControllerAdvice` non espone stack trace
- [ ] Dipendenze aggiornate: Dependabot / Renovate per CVE
- [ ] File upload: limita dimensione, valida tipo, salva fuori dal context path

---

## 13. Checklist Performance

- [ ] Connessioni DB: pool ≤ 15 per istanza
- [ ] Fetch LAZY di default, JOIN FETCH solo dove serve
- [ ] Cache per letture frequenti (Caffeine, Redis)
- [ ] Query N+1 identificate e risolte (usa JPA Query Logging)
- [ ] Paginazione su tutte le list API
- [ ] Compressione Gzip delle risposte (`server.compression.enabled: true`)
- [ ] Log asincrono in produzione (Logback AsyncAppender)
- [ ] Thread pool separato per operazioni bloccanti (@Async)
- [ ] Image optimization (WebP, resize server-side)
- [ ] Static assets serviti da CDN o nginx, non da Spring Boot
- [ ] `spring.jpa.open-in-view` disabilitato per evitare LazyInitializationException (e performance migliori)

---

## 14. Convenzioni Generali

### 14.1 Naming

| Cosa | Convenzione | Esempio |
|---|---|---|
| Classi | PascalCase | `UserService`, `JwtFilter` |
| Metodi | camelCase | `findByEmail()`, `deleteAllExpired()` |
| Costanti | UPPER_SNAKE | `DEFAULT_PAGE_SIZE`, `ROLE_USER` |
| Package | lowercase | `it.president.taskmngr.config.security` |
| Enum valori | UPPER_SNAKE | `USER`, `ADMIN`, `PENDING` |
| Colonne DB | snake_case | `owner_id`, `created_at` |
| Colonne @Column | esplicito | `@Column(name = "owner_id")` |
| Endpoint | plurale, kebab | `/api/users`, `/api/tasks/{id}/comments` |
| DTO Request | *Request | `CreateUserRequest` |
| DTO Response | *Response | `UserResponse` |
| Mapper | *Mapper | `UserMapper` (o MapStruct) |

### 14.2 Struttura file Java

Ordine di campi e metodi in una classe:

```
1. constanti statiche (public → private)
2. campi statici
3. campi istanza (private, final prima)
4. costruttori
5. factory statici
6. metodi pubblici
7. metodi protected
8. metodi private
9. getter/setter
10. equals/hashCode/toString
```

### 14.3 Commenti e JavaDoc

- JavaDoc su **classi pubbliche e metodi pubblici** (spiega comportamento, non "setter di x")
- **Niente** commenti su implementazioni ovvie (`// getter`, `// loop`)
- `@param` e `@return` quando aggiungono valore
- `@throws` per eccezioni checked e unchecked importanti

```java
/**
 * Trova utente per email, inclusi i task associati.
 * Usa JOIN FETCH per evitare N+1 sul lazy loading dei task.
 *
 * @param email email dell'utente (case-sensitive)
 * @return User con tasks già caricati, o empty se non trovato
 * @throws IllegalArgumentException se email è null
 */
public Optional<User> findByEmailWithTasks(String email) {
    // ...
}
```

---

## Riferimenti

- [[BE-NOTES/Java/Spring/Security/SecurityConfig e Filter Chain]]
- [[BE-NOTES/Java/Spring/Security/Authorities e RBAC]]
- [[BE-NOTES/Java/Spring/Core/Dependency Injection]]
- [[BE-NOTES/Java/Spring/Core/Bean e Application Context]]
- [[BE-NOTES/Java/Spring/Core/Service Layer]]
- [[Database/SQL/Fondamenti SQL]]
