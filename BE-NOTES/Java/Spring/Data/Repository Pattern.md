---
topic: "Repository Pattern"
parent: "[[BE-NOTES/Java/Spring/Data/Spring Data JPA|Spring Data JPA]]"
nav_prev: "[[Spring Data JPA.md]]"
nav_next: "[[Specifications Dinamiche.md]]"
---


Il Repository è l'interfaccia tra il layer di servizio e il database. Spring Data JPA fornisce **implementazioni automatiche** — scrivi solo l'interfaccia, il codice concreto è generato al volo.

## Quando serve un repository

- Per ogni **entità JPA** che necessita di operazioni CRUD o query personalizzate
- Per separare la logica di business dai dettagli di persistenza
- Per ottenere query derivation, paginazione e specifiche gratuitamente

## Gerarchia delle interfacce

```java
Repository<T, ID>                  // marcatore (nessun metodo)
  └── CrudRepository<T, ID>        // save(), findById(), findAll(), deleteById()
        └── PagingAndSortingRepository<T, ID>  // findAll(Pageable), findAll(Sort)
              └── JpaRepository<T, ID>         // + flush(), saveAndFlush(), deleteInBatch()
```

Nella pratica estendi sempre `JpaRepository` —include tutto il resto.

## Query Derivation — query dal nome del metodo

```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Long> {

    // Derivazione: "findBy" + "User" + "Id"
    List<Task> findByUserId(Long userId);

    // Con paginazione inclusa
    Page<Task> findByStatus(Status status, Pageable pageable);

    // Con condizione composta AND
    Optional<Task> findByUserIdAndTitle(Long userId, String title);

    // Esistenza
    boolean existsByUserIdAndTitle(Long userId, String title);

    // Conteggio
    long countByStatus(Status status);

    // Con ordinamento
    List<Task> findByUserIdOrderByCreatedAtDesc(Long userId);
}
```

Spring Data analizza il nome del metodo e costruisce la query JPQL corrispondente. Supporta: `And`, `Or`, `Between`, `LessThan`, `GreaterThan`, `Like`, `In`, `IgnoreCase`, `OrderBy`.

**Quando usare derivazione:** query semplici, poche condizioni stabili.
**Quando evitarla:** query con 5+ parametri (nome metodo illeggibile), filtri dinamici, JOIN complessi.

## @Query — query personalizzate

```java
@Query("SELECT t FROM Task t WHERE t.user.id = :userId AND t.status = :status")
List<Task> findByUserAndStatus(@Param("userId") Long userId, @Param("status") Status status);

@Query("SELECT t FROM Task t JOIN FETCH t.user WHERE t.id = :id")
Optional<Task> findByIdWithUser(@Param("id") Long id);
    // JOIN FETCH carica la relazione subito (evita LazyInitializationException)
```

**Quando usare `@Query`:**
- Query complesse che la derivazione non può esprimere
- `JOIN FETCH` per forzare il caricamento di relazioni lazy
- `UPDATE`/`DELETE` bulk con `@Modifying`
- Query native SQL con `nativeQuery = true`

## Query native (solo per casi estremi)

```java
@Query(value = "SELECT * FROM tasks WHERE user_id = :uid AND status = :status",
       nativeQuery = true)
List<Task> findByUserNative(@Param("uid") Long uid, @Param("status") String status);
```

**Attenzione:** le query native sono legate allo specifico DB (PostgreSQL). Usale solo per funzionalità DB-specifiche (ES. full-text search, window functions).

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Nome metodo di derivazione troppo lungo | Illeggibile, difficile da mantenere | `findByUserEmailAndStatusAndCreatedAtBefore` | Usa `@Query` con JPQL per query con 3+ condizioni |
| Query derivation per filtri dinamici | Combinazioni esplodono: 4 filtri = 16 metodi | La derivazione non supporta condizioni opzionali | Usa `JpaSpecificationExecutor` + Specifications |
| `@Query` con nomi parametro non corrispondenti | `No parameter registered` o parametro null | `:paramName` nel JPQL ma `@Param("altroNome")` nel metodo | Allinea `:nome` in JPQL con `@Param("nome")` nel metodo |
| `@Modifying` dimenticato su UPDATE/DELETE | `QueryExecutionRequestException` | Spring Data rileva operazioni di modifica senza `@Modifying` | Aggiungi `@Modifying` + `@Transactional` su query di modifica |
| Transazione mancante per query di modifica | `javax.persistence.TransactionRequiredException` | `save()`, `delete()` require transazione attiva | Aggiungi `@Transactional` sul metodo del service |
| Repository che accetta/restituisce entità JSON | Ciclo infinito se ci sono relazioni bidirezionali | Jackson serializza l'entità con tutte le relazioni | Usa DTO nel controller, mai entità direttamente |

## In TaskMngr

| Repository | Query principali |
|---|---|
| `UserRepository` | `findByEmail(String)`, `existsByEmail(String)` |
| `TaskRepository` | `findByUserId(Long)`, filtro per status, paginazione, specifiche per visibilità |
| `TeamRepository` | CRUD base + lock pessimistico per update |
| `TeamMemberRepository` | `findByTeamId(Long)`, `findByUserId(Long)` |

- Query derivation per filtri semplici (1-2 parametri)
- [[BE-NOTES/Java/Spring/Data/Specifications Dinamiche|JpaSpecificationExecutor]] per il sistema di visibilità dei task (filtri dinamici combinatori)
- [[BE-NOTES/Java/Spring/Data/Lock Ottimistico e Pessimistico|Lock pessimistico]] su `TeamRepository` per update concorrenti
