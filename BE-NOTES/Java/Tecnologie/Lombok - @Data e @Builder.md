---
topic: "Lombok — riduzione boilerplate"
parent: "[[BE-NOTES/Java/Tecnologie/Java|Java]]"
nav_prev: "[[Optional e Gestione Null.md]]"
---


Lombok genera codice boilerplate (getter, setter, costruttori, builder, logger) tramite annotazioni, **in fase di compilazione**. Il codice generato è invisibile, ma esiste nel `.class`. In [[TaskMngr]] è usato per DTO, classi di configurazione e talvolta entità.

## Quando usarlo

Lombok è utile quando scrivere manualmente getter/setter/costruttori diventa rumoroso:

- **DTO e classi di trasferimento dati** — `@Data`, `@Builder` per oggetti con molti campi
- **Classi di configurazione** — `@Setter`, `@Getter` per property bindings con `@ConfigurationProperties`
- **Builder per oggetti complessi** — `@Builder` preferibile a costruttori telescopici (5+ parametri)
- **Logger veloce** — `@Slf4j` anziché dichiarare manualmente `LoggerFactory.getLogger(...)`

**Quando NON usarlo:**
- **Entità JPA con relazioni lazy** — `@Data` genera `toString()`, `equals()`, `hashCode()` che caricano tutte le relazioni (N+1 garantito). Preferire `@Getter`/`@Setter` espliciti
- **Record Java** — i [[BE-NOTES/Java/Tecnologie/Java Records|records]] offrono gli stessi benefici senza Lombok
- **Classi con logica nel costruttore** — Lombok sovrascrive i costruttori generati

## Annotazioni principali

```java
@Data                                   // @Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor
@Builder                                // Builder fluido: TaskDto.builder().id(1L).title("abc").build()
@AllArgsConstructor                     // new TaskDto(id, title, status)
@NoArgsConstructor                      // new TaskDto()
@RequiredArgsConstructor                // costruttore solo per campi final/@NonNull
@Slf4j                                  // log (logger SLF4J statico)
@Value                                  // come @Data ma campi final (immutabile)
@With                                   // genera withX(val) → copia con campo modificato
```

## Esempio

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class TaskDto {
    private Long id;
    private String title;
    private Status status;
}

// Uso
TaskDto dto = TaskDto.builder()
    .title("Implementare login")
    .status(Status.TODO)
    .build();
```

## Il problema di @Data sulle entità JPA

Se metti `@Data` su un'entità con relazioni lazy:

```java
@Entity
@Data                                   // ❌ PROBLEMA
public class Task {
    @ManyToOne(fetch = FetchType.LAZY)
    private User user;
}
```

`@Data` genera `toString()` che accede a `user.getEmail()` fuori dalla transazione → `LazyInitializationException`. Anche `equals()` e `hashCode()` caricano tutte le relazioni.

**Soluzione in TaskMngr:** usare solo `@Getter`/`@Setter` sulle entità e gestire manualmente `toString()` con solo campi semplici.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| `@Data` su entità JPA con relazioni lazy | `LazyInitializationException` fuori dalla transazione | `@Data` genera `toString()`, `equals()`, `hashCode()` che accedono a tutte le relazioni | Usa solo `@Getter`/`@Setter` sulle entità, non `@Data` |
| `@Builder` + `@NoArgsConstructor` su stessa classe | Builder genera campi null (aggira i vincoli del costruttore) | `@Builder` usa il costruttore con tutti i parametri; `@NoArgsConstructor` crea istanze senza campi | Usa `@Builder` e `@AllArgsConstructor` insieme; evita `@NoArgsConstructor` sui builder |
| `@Slf4j` su enum | Errore di compilazione | Lombok non può generare campi statici non final su enum | Dichiara il logger manualmente sulle enum |
| `@EqualsAndHashCode` su entità JPA | Loop infinito in bidirezionale, performance degradata | Lazy loading carica tutta la catena di relazioni | Escludi le relazioni: `@EqualsAndHashCode(exclude = {"user", "tasks"})` |
| Usare `@Value` su classe che sarà proxyzata da Spring | Proxy fallisce, comportamenti inaspettati | `@Value` rende campi `final`, Spring AOP non può sovrascrivere | Usa `@Getter` su classe normale per bean Spring |
| `@Builder` non funziona con Jackson deserialization | Deserializzazione JSON fallisce | Jackson ha bisogno di costruttore vuoto o `@JsonDeserialize` | Aggiungi `@NoArgsConstructor`, `@AllArgsConstructor`, `@Jacksonized` (Lombok 1.18.14+) |

## Attenzione ai conflitti con Records

Con Java 17+ i records nativi coprono molti casi in cui prima serviva `@Data` o `@Value`. Regola pratica:

| Scenario | Scelta migliore |
|---|---|
| DTO immutabile | Record Java |
| DTO mutabile (pochi campi) | `@Data` Lombok |
| Builder con molti campi opzionali | `@Builder` Lombok |
| Entità JPA | `@Getter`/`@Setter` Lombok |
| Logger | `@Slf4j` Lombok |
