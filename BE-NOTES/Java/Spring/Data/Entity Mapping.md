---
topic: "Entity Mapping — JPA"
parent: "[[BE-NOTES/Java/Spring/Data/Spring Data JPA|Spring Data JPA]]"
nav_prev: "[[Specifications Dinamiche.md]]"
nav_next: "[[Hibernate/ORM.md]]"
---


Le entità JPA in [[TaskMngr]] sono classi Java che mappano righe delle tabelle PostgreSQL. Ogni entità è un ponte tra il mondo relazionale (tabelle, colonne, FK) e quello object-oriented (oggetti, attributi, riferimenti).

## Quando creare un'entità

Creiamo un'entità per ogni **concetto di dominio che persiste nel database**:
- Entità principali: `Task`, `User`, `Team`, `TeamMember`
- Entità di supporto: `LinkedAccount`, `Avatar`
- Ogni entità ha una tabella dedicata, una chiave primaria e relazioni con altre entità

**Quando NON creare un'entità:**
- Dati temporanei o di calcolo (usa `@SqlResultSetMapping` o DTO)
- Value objects semplici (usa `@Embeddable`)
- Configurazioni (usa `@ConfigurationProperties`)

## Annotazioni fondamentali

```java
@Entity
@Table(name = "tasks", indexes = {
    @Index(name = "idx_tasks_user", columnList = "user_id"),
    @Index(name = "idx_tasks_status", columnList = "status")
})
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "title", nullable = false, length = 255)
    private String title;

    @Enumerated(EnumType.STRING)     // salva come 'TODO', 'IN_PROGRESS', 'DONE'
    private Status status;
}
```

| Annotazione | Cosa fa | Quando serve |
|---|---|---|
| `@Entity` | Dichiara che questa classe è un'entità JPA | Sempre |
| `@Table(name, indexes)` | Specifica tabella e indici DB | Per nominare la tabella e ottimizzare query |
| `@Id` + `@GeneratedValue` | Chiave primaria auto-generata | Sempre (usa IDENTITY per PostgreSQL) |
| `@Column(name, nullable, length)` | Mapping colonna | Per forzare vincoli DB |
| `@Enumerated(STRING)` | Salva enum come stringa | Leggibile, permette rinominare enum |
| `@Column(updatable = false)` | Campo immutabile dopo creazione | `createdAt`, `createdBy` |

## Relazioni tra entità

```java
@Entity
public class Task {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;                       // ogni task appartiene a un utente

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;                       // task può appartenere a un team (opzionale)
}
```

| Relazione | Uso tipico | Fetch default |
|---|---|---|
| `@ManyToOne` | N task → 1 utente | `EAGER` **(cambia in LAZY!)** |
| `@OneToMany(mappedBy)` | 1 team → N membri | `LAZY` |
| `@ManyToMany` | Utenti ↔ Ruoli | `LAZY` |

**Regola d'oro:** usa `FetchType.LAZY` su TUTTE le relazioni. `EAGER` carica tutto subito, anche ciò che non serve, causando N+1 e memoria sprecata. Se serve il dato, caricalo esplicitamente con `JOIN FETCH` o `@EntityGraph`.

## Indici — quando e perché

```java
@Table(name = "tasks", indexes = {
    @Index(name = "idx_tasks_user", columnList = "user_id"),
    @Index(name = "idx_tasks_status", columnList = "status")
})
```

Gli indici accelerano `WHERE`, `ORDER BY` e `JOIN`. Metti indici su:
- Colonne usate in `findBy*()` — JPA li usa automaticamente
- Chiavi esterne (`user_id`, `team_id`) — JOIN più veloci
- Colonne di ordinamento frequenti (`created_at`, `status`)

**Non mettere** indici su colonne con bassa selettività (es. booleani) — non vengono usati.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `@Data` di Lombok su entità | `LazyInitializationException` fuori transazione | `toString()`, `equals()`, `hashCode()` caricano relazioni lazy | Usa solo `@Getter`/`@Setter` sulle entità |
| `FetchType.EAGER` su relazioni | N+1 query, performance imprevedibili | `@ManyToOne(fetch = EAGER)` è il default JPA | Cambia sempre in `LAZY`; carica esplicitamente con `JOIN FETCH` |
| ID non generato automaticamente | Violazione PK o errore a runtime | Ogni entità deve avere un ID univoco | Usa `@GeneratedValue(strategy = IDENTITY)` e tipo `Long` |
| Enum salvato come `ORDINAL` | Dati corrotti se riordini i valori enum | `@Enumerated(ORDINAL)` salva la posizione numerica | Usa sempre `@Enumerated(STRING)` |
| Dimenticare `@Column(name = "...")` | Nome colonna camelCase in DB (es. `createdAt` invece di `created_at`) | Default Hibernate usa il nome del campo Java | Sii esplicito con `@Column(name = "snake_case")` |
| Mappare una FK manualmente invece di usare relazione | JOIN manuali, nessuna navigazione oggettuale | "Tanto funziona" | Usa `@ManyToOne` + `@JoinColumn` per relazioni |
| Costruttore vuoto assente | `PersistenceException` a runtime | JPA richiede costruttore vuoto (non serve se ci sono altri costruttori) | Aggiungi costruttore `protected` senza argomenti |

## Best practice

- `FetchType.LAZY` sempre e comunque
- `Long` come tipo ID (serializzabile per distribuzione, compatibile con tutte le DB)
- **Mai `@Data`** di Lombok sulle entità — usa `@Getter`/`@Setter` espliciti (previene LazyInitializationException)
- `@Column(updatable = false)` su campi che non devono cambiare mai (createdAt, createdBy)
- Usa `@EntityListeners(AuditingEntityListener.class)` per [[BE-NOTES/Java/Spring/Data/JPA Auditing|auditing automatico]]
