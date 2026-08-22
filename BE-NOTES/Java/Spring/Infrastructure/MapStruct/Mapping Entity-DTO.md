---
topic: "Mapping Entity-DTO"
parent: "[[BE-NOTES/Java/Spring/Infrastructure/MapStruct/MapStruct|MapStruct]]"
nav_prev: "[[../Docker/Docker.md]]"
nav_next: "[[MapStruct.md]]"
---


In un'applicazione con layer separati (entity → service → controller → response), devi **copiare campi da un oggetto all'altro** continuamente. Farlo a mano è noioso, ripetitivo e fonte di bug (dimentichi un campo, il tipo cambia e non te ne accorgi). MapStruct genera automaticamente il codice di mapping in **compile-time**, con performance identiche al codice scritto a mano.

## Quando usare MapStruct

- **Entity ↔ DTO mapping** — l'uso principale: convertire entità JPA in DTO e viceversa
- **Oggetti con molti campi** — 10+ campi, mapping manuale = centinaia di righe boilerplate
- **Mapping con logica di trasformazione** — concatenare nomi, formattare date, calcolare derivati
- **Mapping tra oggetti di layer diversi** — `TaskCreateRequest → Task`, `Task → TaskDto`

**Quando NON usarlo:**
- Mapping semplici con 1-2 campi (scrivi una riga a mano)
- Conversione tra stesso tipo (non serve)
- Quando usi già [[BE-NOTES/Java/Tecnologie/Java Records|records]] con conversione diretta

## Configurazione Maven

```xml
<annotationProcessorPaths>
    <path>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </path>
    <path>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-processor</artifactId>
    </path>
</annotationProcessorPaths>
```

**Ordine importante:** Lombok processa prima (genera getter/setter), poi MapStruct li usa per il mapping. Se MapStruct processa prima, non trova i getter e fallisce.

## Mapping base

```java
@Mapper(componentModel = "spring")    // genera un bean Spring (iniettabile ovunque)
public interface TaskMapper {

    TaskDto toDto(Task entity);           // genera: new TaskDto(task.getId(), task.getTitle(), ...)

    Task toEntity(TaskCreateRequest request);  // genera: new Task(taskRequest.getTitle(), ...)
}
```

MapStruct abbina i campi per **nome**: se entity ha `title` e DTO ha `title`, li mappa automaticamente. Se i tipi differiscono (`LocalDate → String`), MapStruct cerca di convertirli.

## Mapping con nomi diversi

```java
@Mapper(componentModel = "spring", uses = {UserMapper.class, TeamMapper.class})
public interface TaskMapper {

    @Mapping(target = "userId", source = "user.id")        // Task.user.id → TaskDto.userId
    @Mapping(target = "teamName", source = "team.name")    // Task.team.name → TaskDto.teamName
    @Mapping(target = "createdAt", source = "createdAt", dateFormat = "dd/MM/yyyy")
    TaskDto toDto(Task entity);

    @Mapping(target = "user", ignore = true)   // user viene impostato manualmente nel service
    @Mapping(target = "team", ignore = true)   // team viene impostato manualmente nel service
    Task toEntity(TaskCreateRequest request);
}
```

## Mapping di oggetti annidati (uses)

```java
@Mapper(componentModel = "spring", uses = UserMapper.class)
public interface TaskMapper {
    TaskDto toDto(Task entity);
    // MapStruct sa che Task.user → UserDto, e chiama automaticamente UserMapper.toDto()
}

@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDto toDto(User entity);
    // List<Team>? MapStruct chiama automaticamente TeamMapper se specificato in uses
}
```

Evita di re-inventare la ruota ogni volta — `uses` compone i mapper automaticamente.

## @AfterMapping — logica post-mappatura

Per trasformazioni che non sono semplici copie:

```java
@Mapper(componentModel = "spring")
public abstract class TaskMapper {
    // Classe astratta invece di interfaccia per metodi default

    public abstract TaskDto toDto(Task entity);

    @AfterMapping
    protected void setDisplayName(Task entity, @MappingTarget TaskDto dto) {
        // Imposta un campo calcolato dopo il mapping automatico
        dto.setDisplayName(entity.getTitle() + " (" + entity.getStatus() + ")");
    }
}
```

## In TaskMngr

| Mapper | Mapping | Note |
|---|---|---|
| `TaskMapper` | Task ↔ TaskDto ↔ Request DTO | `uses = {UserMapper, TeamMapper}` |
| `UserMapper` | User ↔ UserDto | Mapping di linked accounts e avatar |
| `TeamMapper` | Team ↔ TeamDto | Mapping membri del team |
| `LinkedAccountMapper` | LinkedAccount ↔ Dto | Semplice campi diretti |
| `AvatarMapper` | Avatar ↔ Dto | Mapping multi-sorgente |

- `componentModel = "spring"` sempre (iniettabile nei service)
- `unmappedTargetPolicy = ReportingPolicy.WARN` — avverte se un campo non è mappato
- Target DTO sono [[BE-NOTES/Java/Tecnologie/Java Records|records]] immutabili
- `@AfterMapping` per campi derivati (es. nome composto)

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Lombok processa dopo MapStruct | `Can't map property "X"` | Maven non rispetta l'ordine degli annotation processor | Metti `lombok` PRIMA di `mapstruct-processor` in `annotationProcessorPaths` |
| Campo non mappato | Il target DTO ha campo `null` | Nome campo diverso tra sorgente e target, o mapping mancante | Aggiungi `unmappedTargetPolicy = ReportingPolicy.WARN` per vedere i warning |
| `uses` mancante per oggetti annidati | MapStruct non mappa automaticamente `User → UserDto` | L'oggetto annidato non ha un mapper referenziato | Aggiungi `uses = {UserMapper.class}` nell'annotazione `@Mapper` |
| Ciclo infinito nel mapping | `StackOverflowError` | A → B, B → A (bidirezionale) senza interruzione | Usa `@Mapping(target = "user", ignore = true)` su un lato |
| `@AfterMapping` non eseguito | Metodo post-mappatura ignorato | Firma del metodo errata (parametri sbagliati) | Usa firma corretta: `void method(Source s, @MappingTarget Target t)` |
| `componentModel = "spring"` mancante | Mapper non trovato via injection | Mapper non è un bean Spring | Aggiungi `componentModel = "spring"` nel `@Mapper` |
| Campo immutabile su target record | Errore compilazione: no setter | MapStruct non può mappare su record immutabili senza costruttore | Usa `@Mapping` con `target = "nome"` e record con costruttore all-args |
