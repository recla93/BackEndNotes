---
topic: "Specifications Dinamiche"
parent: "[[BE-NOTES/Java/Spring/Data/Spring Data JPA|Spring Data JPA]]"
---

# Specifications Dinamiche

Le specifiche risolvono un problema comune: **filtri che cambiano in base all'input dell'utente**. Con `JpaSpecificationExecutor` costruisci query WHERE dinamicamente, unendo condizioni solo se necessario, senza scrivere `@Query` per ogni combinazione.

## Quando usarle

- **Ricerca avanzata** — l'utente sceglie su quali campi filtrare (es. "filtra per status + userId + data")
- **Filtri obbligatori + opzionali** — alcune condizioni sono sempre attive (es. visibilità), altre opzionali
- **Sistema di permessi** — condizioni di sicurezza applicate a ogni query (es. "l'utente vede solo i propri task")
- **Combinazioni variabili** — 4 filtri opzionali = 16 combinazioni possibili; scrivere 16 query `@Query` è impossibile

**Quando NON usarle:**
- Query fisse con 1-2 condizioni (basta query derivation)
- Query che coinvolgono sempre STESSE tabelle con STESSE condizioni (usa `@Query`)

## Configurazione

```java
public interface TaskRepository extends JpaRepository<Task, Long>,
                                        JpaSpecificationExecutor<Task> { }
```

JpaSpecificationExecutor aggiunge: `findAll(Specification)`, `findAll(Specification, Pageable)`, `findAll(Specification, Sort)`, `count(Specification)`.

## Costruire specifiche

```java
public class TaskSpecifications {

    // Condizione semplice: uguaglianza
    public static Specification<Task> byUserId(Long userId) {
        return (root, query, cb) ->
            cb.equal(root.get("user").get("id"), userId);
    }

    // Condizione su enum
    public static Specification<Task> byStatus(Status status) {
        if (status == null) return Specification.where(null);
            // se null, non applica il filtro
        return (root, query, cb) ->
            cb.equal(root.get("status"), status);
    }

    // Condizione LIKE (case-insensitive)
    public static Specification<Task> titleContains(String keyword) {
        if (keyword == null || keyword.isBlank()) return Specification.where(null);
        return (root, query, cb) ->
            cb.like(cb.lower(root.get("title")), "%" + keyword.toLowerCase() + "%");
    }
}
```

## Combinare specifiche

```java
// Filtro dinamico: solo condizioni con valore vengono applicate
Specification<Task> spec = Specification
    .where(TaskSpecifications.byUserId(request.userId()))
    .and(TaskSpecifications.byStatus(request.status()))
    .and(TaskSpecifications.titleContains(request.keyword()));

// Con paginazione e ordinamento
Page<Task> tasks = taskRepository.findAll(spec, pageable);
```

Se ad esempio `request.keyword()` è null, quella condizione viene ignorata — la query finale avrà solo `WHERE user_id = ? AND status = ?`.

## Esempio concreto: sistema di visibilità di TaskMngr

```java
public class TaskVisibilitySpecs {

    // L'utente vede i task se:
    // - li ha creati lui (createdBy = userId)
    // - appartiene al team del task
    // - ha un ruolo che gliene dà diritto
    public static Specification<Task> visibleBy(Long userId, Set<Long> teamIds) {
        return (root, query, cb) -> {
            Predicate owned = cb.equal(root.get("createdBy"), userId);
            Predicate inTeam = root.get("team").get("id").in(teamIds);
            return cb.or(owned, inTeam);
        };
    }
}
```

Il service chiama:
```java
Specification<Task> spec = Specification
    .where(TaskVisibilitySpecs.visibleBy(userId, userTeamIds))
    .and(TaskSpecifications.byStatus(filters.status()))
    .and(TaskSpecifications.titleContains(filters.keyword()));

return taskRepository.findAll(spec, pageable).map(mapper::toDto);
```

Una `Specification` combinata fa tutto in una query SQL — nessun N+1, nessuna logica sparsa nei service.

## Attenzione a JOIN impliciti

```java
(root, query, cb) -> cb.equal(root.get("user").get("id"), userId);
// Questo genera: WHERE user.id = ? (JOIN implicito tra task e user)
```

Se usi `root.get("team").get("members").get("id")` attraversi relazioni — assicurati che ci siano JOIN FETCH o index appropriati per non degradare le performance.
