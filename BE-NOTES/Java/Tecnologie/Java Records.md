---
topic: "Java Records — classi dati immutabili"
parent: "[[BE-NOTES/Java/Tecnologie/Core Concepts|Core Concepts]]"
nav_prev: "[[Core Concepts.md]]"
nav_next: "[[Java 21 - Pattern Matching.md]]"
---


I **records** (introdotti in Java 16, standardizzati in Java 17) sono un modo compatto per dichiarare classi che sono contenitori trasparenti di dati immutabili. In [[TaskMngr]] sono la scelta principale per DTO di risposta e richiesta.

## Quando usarli

I records sono ideali per **classi che esistono solo per trasportare dati**, senza comportamenti complessi:

- **DTO di risposta API** — dati da restituire al client, mai modificati dopo la creazione (`UserDto`, `TaskDto`)
- **DTO di richiesta** — dati ricevuti dal client e passati al service layer
- **Value objects** — `Email`, `PhoneNumber`, `Money` (immutabili per natura, validati alla creazione)
- **Chiavi composite** — classi usate come `@IdClass` in JPA
- **Record annidati** per risposte composite: `ApiResponse<List<TaskDto>>`

**Quando evitarli:**
- **Entità JPA** — JPA richiede proxy (record è `final`), costruttore vuoto, `setter` per lazy loading
- **Classi con molti comportamenti** — se il codice opera sui dati interni, meglio una classe normale
- **Classi che devono estendere altre classi** — i records sono `final`, non si possono estendere (ma possono implementare interfacce)

## Cosa generano automaticamente

```java
public record UserDto(Long id, String name, String email) { }
```

Questa singola riga genera:
- **Costruttore canonico**: `new UserDto(id, name, email)` — assegna ogni parametro al componente corrispondente
- **Accessor**: `userDto.id()` invece di `userDto.getId()` — convenzione diversa da JavaBean
- **`equals()` e `hashCode()`** — basati su TUTTI i componenti, utili per confronto valori
- **`toString()`** — stampa tutti i componenti, utile per logging

Differenza chiave rispetto a una classe con Lombok `@Data`:
| Caratteristica | Record | @Data Lombok |
|---|---|---|
| Immutabilità | Sì (componenti `final`) | No (setter generati) |
| Ereditarietà | No (classe `final`) | Sì |
| Accessor | `field()` | `getField()` (convention JavaBean) |
| Boilerplate | Zero righe | `@Data` su classe |

## Compact Constructor

Puoi aggiungere validazione nel costruttore senza doverne scrivere uno manuale completo:

```java
public record Email(String value) {
  public Email {
    if (value == null || !value.contains("@")) {
      throw new IllegalArgumentException("Email non valida: " + value);
    }
    // value è già assegnato implicitamente, non serve this.value = value
  }
}
```

A differenza di un costruttore normale, il compact constructor **non richiede** di assegnare i campi — avviene automaticamente dopo il blocco. Serve solo per validazione, normalizzazione o log.

## Record Patterns (Java 21)

I record patterns permettono di **decomporre un record direttamente** in un pattern match, senza chiamare gli accessor:

```java
if (obj instanceof UserDto(var id, var name)) {
  System.out.println("Utente " + id + ": " + name);
  // id e name sono già variabili locali, cast + accessor automatici
}

// Con switch pattern matching (Java 21)
return switch (response) {
  case ApiResponse.Success(var data) -> processa(data);
  case ApiResponse.Error(var msg, var errors) -> gestisciErrore(msg, errors);
  case null -> "Nessuna risposta";
};
```

## Uso con MapStruct

I records sono un target eccellente per [[BE-NOTES/Java/Spring/Infrastructure/MapStruct/MapStruct|MapStruct]] perché sono immutabili — il mapper genera codice che usa il costruttore canonico o il builder se annotato con `@Builder`.

```java
// TaskMapper.java
@Mapper(componentModel = "spring")
public interface TaskMapper {
    TaskDto toDto(Task entity);
    // MapStruct genera: new TaskDto(task.getId(), task.getTitle(), ...)
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Record usato come entità JPA | `HibernateException` o proxy non funzionante | I record sono `final`, non hanno costruttore vuoto, non permettono setter | Usa una classe normale per entità JPA; i record solo per DTO/Value Object |
| Tentativo di estendere un record | Errore di compilazione | I record sono implicitamente `final`, non estendibili | Usa composizione o interfacce; i record possono implementare interfacce ma non estendere classi |
| Accessor con prefisso `get` | Inconsistenza di stile, mappatura Jackson errata | I record generano accessor come `name()` non `getName()` | Jackson (2.12+) supporta gli accessor dei record; se devi retro-compatibilità, usa `@JsonGetter("name")` |
| Compact constructor troppo invasivo | Effetto collaterale inaspettato | Modifichi il parametro invece di validarlo | Il compact constructor è solo per validazione/normalizzazione; non assegnare campi esplicitamente |
| equals/hashCode non adatti al contesto | Due istanze logicamente diverse sono considerate uguali | `equals()` usa TUTTI i componenti del record | Se l'uguaglianza deve ignorare certi campi, non usare record: scrivi una classe normale con equals custom |

## In TaskMngr

- `ApiResponse<T>` è un record generico usato in ogni controller
- Tutte le request e response DTO sono records (<10 righe l'uno)
- Combinati con [[BE-NOTES/Java/Tecnologie/Core Concepts/Java 21 - Pattern Matching|pattern matching]] per dispatch sulle risposte
