---
topic: "JPA Auditing"
parent: "[[BE-NOTES/Java/Spring/Data/Spring Data JPA|Spring Data JPA]]"
---

# JPA Auditing

L'auditing automatico popola i campi `createdAt`, `updatedAt`, `createdBy`, `lastModifiedBy` senza scrivere una riga di logica nei service. Spring Data JPA lo fa tramite **entity listeners** attivati da annotazioni.

## Quando serve l'auditing

Praticamente **sempre** — ogni entità dovrebbe tracciare quando e da chi è stata creata/modificata:
- **Forensic**: sapere chi ha creato/modificato un record
- **Debugging**: capire quando un'entità è stata modificata l'ultima volta
- **Audit esterno**: requisiti normativi (GDPR, SOX)

## Configurazione

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaConfig { }

@Bean
public AuditorAware<Long> auditorProvider() {
    return () -> Optional.ofNullable(SecurityUtils.getCurrentUserId());
    // SecurityUtils estrae l'userId dal SecurityContextHolder
    // Se l'utente non è autenticato (registrazione, batch), restituisce null
}
```

`auditorAwareRef` è cruciale: dice a Spring come ottenere l'utente corrente. Il bean deve restituire un `Optional<Long>` con l'ID dell'utente loggato.

## Uso sulle entità

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Task {

    @CreatedDate
    @Column(updatable = false, nullable = false)
    private Instant createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private Long createdBy;

    @LastModifiedBy
    private Long lastModifiedBy;
}
```

| Annotazione | Cosa fa | Note |
|---|---|---|
| `@CreatedDate` | Imposta la data al primo `persist` | `updatable = false` |
| `@LastModifiedDate` | Aggiorna a ogni `merge`/`update` | Si aggiorna sempre |
| `@CreatedBy` | Imposta l'utente creatore | `updatable = false` |
| `@LastModifiedBy` | Aggiorna l'utente modifica | Si aggiorna sempre |

**`@EntityListeners(AuditingEntityListener.class)`** è obbligatorio — senza, le annotazioni vengono ignorate.

## In TaskMngr — BaseEntity

Per evitare ripetizioni, tutte le entità estendono una classe base:

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false, nullable = false)
    private Instant createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private Instant updatedAt;
}

// Uso
@Entity
public class Task extends BaseEntity { ... }
@Entity
public class User extends BaseEntity { ... }
```

## Perché `Instant` e non `LocalDateTime`

- `Instant` è un timestamp UTC — funziona correttamente in fusi orari diversi
- `LocalDateTime` dipende dal fuso orario del server — causa bug quando si cambia regione
- Con `Instant`, il frontend converte nel fuso orario locale del client

## Alternativa: auditing manuale

Potresti impostare i campi manualmente nei service:
```java
task.setCreatedAt(Instant.now());
task.setUpdatedAt(Instant.now());
```

Ma è **ripetitivo** (ogni metodo deve ricordarsi di farlo), **fragile** (facile dimenticare) e **viola DRY**. L'auditing automatico è sempre preferibile.
