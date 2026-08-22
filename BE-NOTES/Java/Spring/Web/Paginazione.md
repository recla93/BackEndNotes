---
topic: "Paginazione"
parent: "[[BE-NOTES/Java/Spring/Web/REST API Design|REST API Design]]"
nav_prev: "[[Global Exception Handler.md]]"
---


Restituire TUTTI i record di una tabella in una sola risposta è sbagliato: dati troppi, memoria sprecata, tempi di risposta lunghi. La paginazione divide il risultato in pagine. Spring Data la supporta nativamente.

## Quando serve

- **Liste lunghe** — task, utenti, team (decine/migliaia di record)
- **Mobile** — bandwidth limitato, schermi piccoli (pagine da 10-20 elementi)
- **Performance** — query con `LIMIT/OFFSET` sono più veloci di `SELECT * senza LIMIT`
- **UX** — scroll infinito o paginazione classica

**Quando NON serve:**
- Risultati di ricerca specifica (pochi record)
- Endpoint di dettaglio (`GET /tasks/123`)
- Dropdown/select con pochi elementi (< 50, carica tutto)

## Controller

```java
@GetMapping
public ResponseEntity<ApiResponse<Page<TaskDto>>> getAll(
        @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
        @ParameterObject                                  // SpringDoc: documenta i parametri
        Pageable pageable) {

    Page<TaskDto> tasks = taskService.findAll(filters, pageable);
    return ResponseEntity.ok(ApiResponse.success(tasks));
}
```

Spring Boot espone automaticamente i parametri `?page=0&size=10&sort=createdAt,desc`.

## Parametri query

| Parametro | Default | Esempio | Cosa fa |
|---|---|---|---|
| `page` | 0 | `?page=2` | Terza pagina (0-based) |
| `size` | 20 | `?size=10` | 10 elementi per pagina |
| `sort` | — | `?sort=createdAt,desc` | Ordina per data decrescente |
| `sort` multipli | — | `?sort=status,asc&sort=createdAt,desc` | Ordina per status, poi data |

## Service

```java
public Page<TaskDto> findAll(TaskFilter filters, Pageable pageable) {
    Specification<Task> spec = buildSpecification(filters);
    return taskRepository.findAll(spec, pageable)
        .map(mapper::toDto);              // mappa ogni elemento → evita loop manuali
}
```

`.map(mapper::toDto)` trasforma ogni entità in DTO — equivalente a:
```java
// senza .map():
Page<Task> entities = taskRepository.findAll(spec, pageable);
List<TaskDto> dtos = entities.stream()
    .map(mapper::toDto)
    .toList();
return new PageImpl<>(dtos, pageable, entities.getTotalElements());
```

## Risposta JSON

```json
{
  "success": true,
  "message": "Operazione completata",
  "data": {
    "content": [
      { "id": 1, "title": "Task A" },
      { "id": 2, "title": "Task B" }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8,
    "first": true,
    "last": false
  }
}
```

| Campo | Cosa indica | Utile per |
|---|---|---|
| `page` | Pagina corrente (0-based) | Display "Pagina 1 di 8" |
| `size` | Elementi per pagina | Grandezza bottone "Carica altri" |
| `totalElements` | Totale record | Badge "150 task trovati" |
| `totalPages` | Totale pagine | Navigazione a pagine |
| `first` / `last` | Prima/ultima pagina | Disabilitare bottoni |
| `content` | Array degli elementi | Lista vera e propria |

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `page` 1-based invece di 0-based | La prima pagina è vuota, la seconda mostra i primi risultati | Spring Data è 0-based, ma alcuni client mandano `?page=1` per prima pagina | Adatta il client a 0-based o converti nel controller: `pageable.first()` se page > 0 |
| Paginazione senza ordinamento | Ordine casuale tra pagine, stesso elemento su pagine diverse | `ORDER BY` senza campi deterministici | Aggiungi sempre `sort` (almeno per id) per ordinamento stabile tra pagine |
| Page totale per milioni di record | `SELECT COUNT(*)` lentissimo su tabella enorme | `Page` esegue sempre count query | Usa `Slice` invece di `Page` se il conteggio esatto non serve, o implementa keyset pagination |
| Paginazione applicata dopo il mapping | Performance: carichi tutte le entità in memoria | `repository.findAll()` senza Pageable poi fai `.stream().skip().limit()` | Usa Pageable nel repository: `findAll(spec, pageable)` — la query fa LIMIT/OFFSET |
| Limit superato senza controllo | Client può richiedere 10000 elementi per pagina | `@PageableDefault(size = 20)` è un default, non un limite | Aggiungi `@Max(100) int size` o un `PageableHandlerMethodArgumentResolverCustomizer` per limitare size massimo |

## Attenzione a OFFSET su tabelle grandi

`LIMIT 20 OFFSET 40` funziona, ma su milioni di record `OFFSET 100000` diventa lento (PostgreSQL scansiona comunque le righe saltate).

Alternative per tabelle molto grandi:
- **Cursor-based pagination** — `WHERE id > ? ORDER BY id LIMIT 20` (nessun OFFSET)
- **Keyset pagination** — usa `@Slice` invece di `@Page` (non conta il totale)
- **Search After** (Elasticsearch style)

Per TaskMngr (migliaia, non milioni di record), `Page` + `Pageable` è perfetto.

## In TaskMngr

- Paginazione su TUTTI gli endpoint di lista: `GET /api/tasks`, `GET /api/users`, `GET /api/teams`
- Default: 20 elementi, ordinati per `createdAt` decrescente
- Combinata con [[BE-NOTES/Java/Spring/Data/Specifications Dinamiche|Specifications]] per filtri dinamici
- `@ParameterObject` per documentazione OpenAPI automatica
