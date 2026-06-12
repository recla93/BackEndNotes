---
topic: "JPA Best Practices — persistenza robusta e performante"
parent: "[[BE-NOTES/Java/Spring/Data/Spring Data JPA|Spring Data JPA]]"
---

# JPA Best Practices

Regole per modellare la persistenza con JPA/Hibernate in modo **sicuro, performante e manutenibile**. Basato sull'esperienza con [[TaskMngr]] e PostgreSQL.

---

## 1. Entity Design

### 1.1 Entity = POJO semplice, nessuna logica di business

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 100)
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

    public Long getId() { return id; }
    public String getEmail() { return email; }
    // getter, no setter per campi immutabili
}
```

**Regole:**
- Costruttore vuoto `protected` (JPA lo richiede)
- ID con `@GeneratedValue(strategy = IDENTITY)` per PostgreSQL/MySQL, `SEQUENCE` per Oracle
- Campi immutabili solo getter, mai setter
- `@Column(nullable = false)` esplicito su ogni campo obbligatorio
- `@Column(unique = true)` per vincoli di business
- `@Column(length = ...)` per varchar
- `@Enumerated(STRING)` mai `ORDINAL` (si rompe se riordini gli enum)

### 1.2 equals/hashCode — attenzione!

```java
// ❌ Problematico con ID generato
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User)) return false;
    return id != null && Objects.equals(id, ((User) o).id);
}

// ✅ Più sicuro: business key (email è unique)
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User other)) return false;
    return Objects.equals(email, other.email);
}

@Override
public int hashCode() {
    return Objects.hash(email);
}
```

**Perché:** se usi `id` in equals/hashCode e l'ID è generato, un oggetto appena creato (ID=null) ha hash diverso prima e dopo il salvataggio. Risultato: si perde in HashSet/HashMap. Usa una business key immutabile o un UUID generato dall'applicazione.

---

## 2. Relazioni

### 2.1 Fetch LAZY sempre

```java
@ManyToOne(fetch = LAZY)        // default LAZY da JPA 2.0 — va bene
@OneToMany(mappedBy = "x")       // default LAZY — va bene
@OneToOne(fetch = LAZY)         // default EAGER — CAMBIA IN LAZY!
@ManyToMany(fetch = LAZY)       // default LAZY — va bene
```

**Regola d'oro:** se non sei sicuro, usa LAZY. EAGER = carica sempre e comunque, anche quando non serve. Causa performance imprevedibili.

### 2.2 mappedBy su un solo lato

```java
@Entity
public class User {
    @OneToMany(mappedBy = "owner", cascade = ALL, orphanRemoval = true)
    private List<Task> tasks = new ArrayList<>();
}

@Entity
public class Task {
    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "owner_id", nullable = false)
    private User owner;
}
```

`mappedBy` va sul lato **One** (quello che NON possiede la FK). La FK è dichiarata sul lato **Many** con `@JoinColumn`.

### 2.3 Cascade — mai CascadeType.ALL su ManyToMany

```java
// ❌ Pericoloso: DELETE su un utente cancella anche corsi condivisi
@ManyToMany(cascade = ALL)
private List<Course> courses;

// ✅ Sicuro: gestisci la cancellazione manualmente
@ManyToMany
@JoinTable(name = "user_courses")
private List<Course> courses;
```

**Regola:** usa cascade = ALL o PERSIST solo su relazioni dove l'entità figlia vive e muore con il padre (es. `User` → `Task` con `orphanRemoval`). Su ManyToMany e relazioni condivise, nessun cascade.

### 2.4 orphanRemoval=true

```java
@OneToMany(mappedBy = "owner", cascade = ALL, orphanRemoval = true)
private List<Task> tasks = new ArrayList<>();
```

`orphanRemoval = true` fa sì che rimuovendo un elemento dalla lista, Hibernate lo cancelli dal DB. Senza, il task rimane orfano con `owner_id = null`.

---

## 3. Fetch Strategy

### 3.1 N+1 — il problema più comune

```java
// ❌ N+1: 1 query per gli utenti + N query per i task di ogni utente
List<User> users = userRepository.findAll();
for (User u : users) {
    System.out.println(u.getTasks().size());  // lazy load → nuova query!
}
```

**Soluzioni:**

```java
// 1. JOIN FETCH (JPQL)
@Query("SELECT u FROM User u JOIN FETCH u.tasks")
List<User> findAllWithTasks();

// 2. @EntityGraph
@EntityGraph(attributePaths = "tasks")
@Query("SELECT u FROM User u")
List<User> findAllWithTasks();

// 3. @BatchSize (mitigazione, non soluzione)
@OneToMany(mappedBy = "owner")
@BatchSize(size = 20)
private List<Task> tasks;
```

### 3.2 DTO Projection (evita di caricare entità intere)

```java
// ❌ Carica tutta l'entità (tutti i campi)
List<User> users = userRepository.findAll();

// ✅ Projection: solo i campi che servono
public interface UserSummary {
    Long getId();
    String getEmail();
    String getName();
}

@Query("SELECT u.id AS id, u.email AS email, u.name AS name FROM User u")
List<UserSummary> findSummaries();
```

### 3.3 open-in-view — DISABILITALO in produzione

```yaml
spring:
  jpa:
    open-in-view: false  # default è TRUE!
```

**Perché:** OSIV mantiene la sessione Hibernate aperta finché la risposta HTTP non è completa. Sembra comodo (nessun LazyInitializationException), ma:
- Tiene la connessione del pool occupata per tutta la durata della richiesta
- Permette di caricare lazy data nel template/serializzazione — maschera il N+1
- Concorrenza ridotta, pool esaurito in produzione

**Cosa fare invece:** carica i dati necessari NEL SERVICE PRIMA di tornare al controller (JOIN FETCH, EntityGraph).

---

## 4. Query

### 4.1 Query Derivation — nomi leggibili

```java
// ❌ Illegibile
List<Task> findByOwnerEmailAndStatusInAndDueDateBefore(String email, List<Status> statuses, LocalDate date);

// ✅ Meglio: @Query + nomi descrittivi
@Query("SELECT t FROM Task t WHERE t.owner.email = :email AND t.status IN :statuses AND t.dueDate < :date")
List<Task> findExpiredTasksByOwner(@Param("email") String email, @Param("statuses") List<Status> statuses, @Param("date") LocalDate date);
```

**Regola:** se il nome supera le 3-4 parole, usa `@Query`. Il nome del metodo deve descrivere COSA fa, non COME lo fa.

### 4.2 Native query — ultima spiaggia

```java
// Preferisci JPQL/HQL
@Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")

// Solo se necessario (feature DB-specifiche: JSONB, full-text, CTE)
@Query(value = "SELECT * FROM users WHERE email ILIKE '%' || :domain", nativeQuery = true)
```

**Regola:** native query = accoppiamento al DB specifico. Se cambi DB (es. H2 in test → PostgreSQL in prod), le native query potrebbero rompersi. Usale solo per feature che JPQL non supporta.

### 4.3 Modifiche in blocco

```java
@Modifying
@Query("UPDATE Task t SET t.status = :status WHERE t.dueDate < :date AND t.status <> 'DONE'")
int updateExpiredTasks(@Param("status") Status status, @Param("date") LocalDate date);
```

`@Modifying` invalida automaticamente la cache di primo livello (ma non quella di secondo livello/Caffeine).

---

## 5. Transaction Management

### 5.1 Transazioni brevi e mirate

```java
// ✅ OK: transazione esplicita sul service
@Service
public class TaskService {

    @Transactional
    public Task create(CreateTaskRequest request) {
        Task task = new Task(request.title(), getCurrentUser());
        return taskRepository.save(task);
    }
}

// ❌ NO: @Transactional su ogni metodo (lock tengono riga più del necessario)
// ❌ NO: @Transactional su metodi di sola lettura senza readOnly=true
```

### 5.2 readOnly per query

```java
@Transactional(readOnly = true)
public List<Task> findAllByOwner(Long ownerId) {
    return taskRepository.findByOwnerId(ownerId);
}
```

`readOnly = true` ottimizza Hibernate (disabilita dirty checking), e alcuni DB impostano transazioni in sola lettura.

---

## 6. Lock e Concorrenza

### 6.1 Lock ottimistico (@Version)

```java
@Entity
public class Task {

    @Version
    private int version;

    // ...
}
```

**Quando usare:** entità modificate concorrentemente da più utenti (es. due admin modificano lo stesso task). Hibernate lancia `OptimisticLockException` se la versione non corrisponde al momento del flush.

**Non usare:** se le collisioni sono frequenti (es. contatore condiviso). In quel caso, lock pessimistico o logica atomica a livello DB.

### 6.2 Lock pessimistico

```java
@Lock(PESSIMISTIC_WRITE)
@Query("SELECT t FROM Task t WHERE t.id = :id")
Optional<Task> findByIdWithLock(@Param("id") Long id);
```

**Quando usare:** operazioni critiche dove due transazioni non possono leggere lo stesso dato contemporaneamente (es. prenotazione ultimo posto).

**Costo:** tiene una riga bloccata nel DB — aumenta il rischio di deadlock.

---

## 7. Performance

### 7.1 Indici

```java
@Entity
@Table(name = "tasks", indexes = {
    @Index(name = "idx_tasks_owner_status", columnList = "owner_id, status"),
    @Index(name = "idx_tasks_due_date", columnList = "due_date")})
public class Task { ... }
```

**Regole:**
- Indice su ogni FK usata in WHERE/JOIN
- Indice composto quando filtri per più colonne (es. `owner_id + status`)
- Indice su colonne ORDER BY / GROUP BY
- NON mettere indici su tutto (ogni indice rallenta INSERT/UPDATE)

### 7.2 Batch fetching

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 20
```

Mitiga N+1: carica più proxy lazy in una volta sola. Non sostituisce JOIN FETCH, ma riduce il danno in caso di lazy load imprevisti.

### 7.3 Connection pool

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      leak-detection-threshold: 60000
```

**Regola:** 10-15 connessioni per microservizio. Più connessioni = più contesa sul DB, non più velocità.

---

## 8. Schema Management

### 8.1 Migrazioni versionate (Flyway)

```yaml
# application-prod.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration

# application-dev.yml
spring:
  jpa:
    hibernate:
      ddl-auto: update
  flyway:
    enabled: false
```

**Regole:**
- MAI `ddl-auto=update` in produzione (può droppare colonne, non versiona)
- Flyway script: `V1__init.sql`, `V2__add_role_column.sql`, etc.
- Gli script DEVONO essere idempotenti o avere controllo su già-eseguiti
- Mai modificare uno script già eseguito — creane uno nuovo

### 8.2 Column naming

```sql
-- PostgreSQL: snake_case per colonne, lowercase per nomi
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

```java
@Column(name = "created_at")
private LocalDateTime createdAt;
```

**Regola:** sii esplicito con `@Column(name = "...")`. Non fidarti della convenzione Hibernate (camelCase → snake_case automatica) — cambia tra versioni.

---

## 9. Anti-pattern JPA

| Anti-pattern | Problema | Soluzione |
|---|---|---|
| `ddl-auto=update` in prod | Perdita dati, nessuna storia | Flyway |
| EAGER fetch | Carica sempre tutto | LAZY + JOIN FETCH |
| N+1 in controller | Query infinite | JOIN FETCH, EntityGraph |
| Entity esposta in API | Coupling API-DB, leak dati | DTO |
| @Transactional su tutto | Lock tenuti troppo a lungo | readOnly=true per letture |
| equals/hashCode basato su ID | Bug con oggetti transient | Business key |
| OSIV=true (default) | Pool esaurito, maschera N+1 | OSIV=false |
| CascadeType.ALL su ManyToMany | Cancellazioni a cascata inaspettate | Nessun cascade |
| `@Data` di Lombok su entity | equals/hashCode problematici | `@Getter @Setter` manuali |
| `@Column(columnDefinition)` | Accoppiamento al DB specifico | `@Column(length=...)`, `@Enumerated` |
| fetch size 1 per default | N+1 per collection lazy | `default_batch_fetch_size` |
| Transazioni giganti | Deadlock, lock contesi | Operazioni piccole e veloci |

---

## 10. Checklist Finale

- [ ] Entity: costruttore vuoto protected, @GeneratedValue, @Column espliciti
- [ ] Fetch: LAZY su tutte le relazioni (anche @OneToOne)
- [ ] N+1: verificato con logging SQL o tool (JPA Buddy, Hibernate Stats)
- [ ] equals/hashCode: basato su business key, non su ID
- [ ] Index: su FK, colonne WHERE/GROUP BY/ORDER BY
- [ ] OSIV: disabilitato in produzione
- [ ] Transaction: piccole, readOnly=true per letture
- [ ] ddl-auto: update solo in dev; Flyway in prod
- [ ] Connection pool: 10-15, leak detection attiva
- [ ] DTO: mai entity in input/output API
- [ ] @Version: su entità concorrenti
- [ ] `@Data`/`@EqualsAndHashCode`: evitato su entity

---

## Riferimenti

- [[BE-NOTES/Java/Spring/Data/Entity Mapping]]
- [[BE-NOTES/Java/Spring/Data/Repository Pattern]]
- [[BE-NOTES/Java/Spring/Data/Lock Ottimistico e Pessimistico]]
- [[BE-NOTES/Java/Spring/Data/Hibernate/Relazioni e Mappature]]
- [[BE-NOTES/Java/Spring/Boot/Spring Boot Best Practices]]
