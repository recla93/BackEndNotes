---
topic: "JPA — Java Persistence API"
nav_prev: "[[ORM.md]]"
nav_next: "[[Hibernate e Session Factory.md]]"
---

**JPA** (Java Persistence API) è lo **standard Java per ORM**. Definisce come gli oggetti vengono persistiti (salvati) nel database.

**Hibernate** è un'implementazione di JPA (ce ne sono altre, ma Hibernate è la più usata).

### Differenza JPA vs Hibernate

| JPA | Hibernate |
|-----|-----------|
| Standard Java | Implementazione di JPA |
| Definisce l'interfaccia | Implementa l'interfaccia |
| Usa `jakarta.persistence` (Java EE) | Usa `org.hibernate` |
| Generico | Specifico per Hibernate |

### API Fondamentali di JPA

- `EntityManager`: gestisce le entità (come Session di Hibernate)
- `EntityManagerFactory`: crea EntityManager
- `Entity`: annota le classi da persistere
- `Query`: esegue query HQL o JPQL

```java
// Creazione di EntityManagerFactory
EntityManagerFactory emf = Persistence.createEntityManagerFactory(
    "mio_database"
);

// Creazione di EntityManager
EntityManager em = emf.createEntityManager();

// Operazioni CRUD
EntityTransaction tx = em.getTransaction();
tx.begin();
try {
    // CREATE
    Persona p = new Persona();
    p.setNome("Mario");
    em.persist(p);
    
    // READ
    Persona mario = em.find(Persona.class, 1);
    
    // UPDATE
    mario.setEta(31);
    em.merge(mario);
    
    // DELETE
    em.remove(mario);
    
    tx.commit();
} catch (Exception e) {
    tx.rollback();
} finally {
    em.close();
}
```

## Annotazioni Hibernate

### @Entity

Marca una classe come **entità persistente** (mapperà a una tabella):

```java
@Entity
@Table(name = "persona")  // Nome della tabella
public class Persona {
    // ...
}
```

### @Id e @GeneratedValue

```java
@Entity
public class Persona {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;  // Chiave primaria, auto-increment
}
```

**Strategie di generazione**:
- `IDENTITY`: delega al DB (AUTO_INCREMENT in MySQL)
- `SEQUENCE`: usa una sequenza DB (PostgreSQL, Oracle)
- `TABLE`: usa una tabella per tracciare i valori
- `AUTO`: Hibernate sceglie automaticamente

### @Column

Specifica il mapping di una colonna:

```java
@Entity
public class Persona {
    @Column(name = "nome_persona", length = 100, nullable = false)
    private String nome;
    
    @Column(unique = true)
    private String email;
}
```

### @Transient

Proprietà che **non** viene salvata nel database:

```java
@Entity
public class Persona {
    @Transient
    private String campoTemporaneo;  // Non va nel DB
}
```

### Relazioni: @OneToMany e @ManyToOne

```java
// Tabella padre
@Entity
public class Regione {
    @Id
    private int id;
    
    @OneToMany(mappedBy = "regione")
    private List<Provincia> province;
}

// Tabella figlia
@Entity
public class Provincia {
    @Id
    private int id;
    
    @ManyToOne
    @JoinColumn(name = "id_regione")  // Chiave esterna
    private Regione regione;
}
```

### Relazioni: @ManyToMany

```java
@Entity
public class Docente {
    @ManyToMany
    @JoinTable(
        name = "docente_materia",  // Tabella associativa
        joinColumns = @JoinColumn(name = "id_docente"),
        inverseJoinColumns = @JoinColumn(name = "id_materia")
    )
    private List<Materia> materie;
}

@Entity
public class Materia {
    @ManyToMany(mappedBy = "materie")
    private List<Docente> docenti;
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Usare Hibernate API invece di JPA API | Accoppiamento a Hibernate, difficile cambiare implementazione | Chiamare `session.save()` invece di `em.persist()` | Usa sempre `jakarta.persistence.*` (EntityManager) non `org.hibernate.*` (Session) |
| `EntityManager` non chiuso | Memory leak, connessione non rilasciata | `em.close()` dimenticato | Chiudi in `finally` o usa `@PersistenceContext` in Spring (gestito automaticamente) |
| `em.merge()` usato per creare nuove entità | SELECT aggiuntiva prima di INSERT | `merge()` fa prima una query per verificare se l'entità esiste | Usa `em.persist()` per nuove entità, `em.merge()` solo per update di entità detached |
| `em.remove()` su entità detached | `IllegalArgumentException` o `EntityNotFoundException` | Rimuovere un'entità non gestita dal persistence context | Usa `find()` prima di `remove()`, o passa `em.merge(entity)` a remove |
| Transazione manuale senza rollback in caso di errore | Dati parziali, DB inconsistente | `tx.rollback()` non chiamato se eccezione | Avvolgi in try-catch e fai rollback nel catch |

### @Transactional

Gestisce le transazioni (tutto o niente):

```java
@Service
public class PersonaService {
    @Transactional
    public void salvaPersone(List<Persona> persone) {
        for (Persona p : persone) {
            repository.save(p);
        }
        // Se una eccezione avviene, tutto viene annullato (rollback)
    }
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Usare Hibernate API invece di JPA API | Accoppiamento a Hibernate, difficile cambiare implementazione | Chiamare `session.save()` invece di `em.persist()` | Usa sempre `jakarta.persistence.*` (EntityManager) non `org.hibernate.*` (Session) |
| `EntityManager` non chiuso | Memory leak, connessione non rilasciata | `em.close()` dimenticato | Chiudi in `finally` o usa `@PersistenceContext` in Spring (gestito automaticamente) |
| `em.merge()` usato per creare nuove entità | SELECT aggiuntiva prima di INSERT | `merge()` fa prima una query per verificare se l'entità esiste | Usa `em.persist()` per nuove entità, `em.merge()` solo per update di entità detached |
| `em.remove()` su entità detached | `IllegalArgumentException` o `EntityNotFoundException` | Rimuovere un'entità non gestita dal persistence context | Usa `find()` prima di `remove()`, o passa `em.merge(entity)` a remove |
| Transazione manuale senza rollback in caso di errore | Dati parziali, DB inconsistente | `tx.rollback()` non chiamato se eccezione | Avvolgi in try-catch e fai rollback nel catch |

### @DiscriminatorColumn (Single Table Inheritance)

Usa una colonna per distinguere i sottotipi:

```java
@Entity
@Table(name = "persona")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo")
@DiscriminatorValue("persona")
public class Persona {
    @Id
    private int id;
    private String nome;
}

@Entity
@DiscriminatorValue("studente")
public class Studente extends Persona {
    private int numeroMatricola;
}

@Entity
@DiscriminatorValue("docente")
public class Docente extends Persona {
    private String stipendio;
}
```