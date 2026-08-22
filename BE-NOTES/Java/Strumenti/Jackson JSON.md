---
topic: "Jackson — JSON Processing"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

Jackson e la libreria standard per serializzazione/deserializzazione JSON in Java. Spring Boot la include automaticamente tramite `spring-boot-starter-web`. Gestisce mapping tra oggetti Java e JSON con annotazioni, supporto a generics e formati aggiuntivi (XML, YAML, CSV).

Alternativa: Gson (Google) e piu leggera ma meno potente. Jackson e de facto standard in ecosistema Spring.

## ObjectMapper — classe centrale

```java
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();

// Java → JSON (serializzazione)
String json = mapper.writeValueAsString(oggetto);

// JSON → Java (deserializzazione)
Oggetto obj = mapper.readValue(json, Oggetto.class);

// Pretty print
String pretty = mapper.writerWithDefaultPrettyPrinter()
    .writeValueAsString(oggetto);

// Da file
Oggetto obj = mapper.readValue(new File("data.json"), Oggetto.class);
```

`ObjectMapper` e thread-safe dopo la configurazione. In Spring Boot, puoi iniettare il bean `ObjectMapper` configurato automaticamente. Per thread-safety, configura una volta e riusa.

## Configurazione base

```java
ObjectMapper mapper = new ObjectMapper();

// Formattazione
mapper.enable(SerializationFeature.INDENT_OUTPUT);        // pretty print
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);  // ISO date
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES); // ignora campi extra
mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);  // snake_case
```

```properties
# Spring Boot: equivalenti in application.properties
spring.jackson.serialization.indent-output=true
spring.jackson.serialization.write-dates-as-timestamps=false
spring.jackson.deserialization.fail-on-unknown-properties=false
spring.jackson.property-naming-strategy=SNAKE_CASE
```

Configurazioni comuni: pretty print, date in ISO-8601, ignorare proprieta sconosciute in input (evita errori quando il client invia campi extra), naming strategy per convertire tra `snake_case` JSON e `camelCase` Java.

## Annotazioni principali

```java
import com.fasterxml.jackson.annotation.*;

public class Utente {

    @JsonProperty("user_id")
    private Long id;

    @JsonIgnore
    private String password;

    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    private LocalDateTime createdAt;

    @JsonFormat(pattern = "dd/MM/yyyy")
    private LocalDate dataNascita;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String note;
}

// Serializzazione condizionale
@JsonIgnoreProperties({"password", "secretKey"})
public class RispostaDTO {
    // ...
}
```

`@JsonProperty` personalizza il nome nel JSON. `@JsonIgnore` esclude campi (password, dati sensibili). `@JsonFormat` controlla il formato delle date. `@JsonInclude(NON_NULL)` esclude campi null dal JSON (riduce dimensione).

## Gestione date

```java
// Globale (ObjectMapper)
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
mapper.registerModule(new JavaTimeModule());  // per java.time.*

// Per singolo campo
public class Evento {
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    private LocalDateTime data;

    @JsonFormat(pattern = "dd/MM/yyyy")
    private LocalDate giorno;
}
```

Jackson non serializza `java.time` (LocalDate, LocalDateTime) senza `JavaTimeModule`. In Spring Boot e registrato automaticamente. Le date escono come array `[2024, 1, 15]` di default; disabilita `WRITE_DATES_AS_TIMESTAMPS` per formato ISO.

## Deserializzazione di oggetti generici

```java
import com.fasterxml.jackson.core.type.TypeReference;

// List<T>
List<Utente> utenti = mapper.readValue(
    jsonArray,
    new TypeReference<List<Utente>>() {}
);

// Map<String, T>
Map<String, Utente> mappa = mapper.readValue(
    jsonMap,
    new TypeReference<Map<String, Utente>>() {}
);

// JsonNode (albero generico, utile per JSON dinamici)
JsonNode root = mapper.readTree(json);
JsonNode nome = root.get("utente").get("nome");
String valore = nome.asText();
```

`TypeReference` preserva i generics a runtime (altrimenti `List.class` perderebbe `<Utente>`). `JsonNode` permette di navigare JSON senza creare classi DTO.

## Custom deserializer/serializer

```java
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;

// Deserializer custom
public class LocalDateDeserializer extends JsonDeserializer<LocalDate> {
    @Override
    public LocalDate deserialize(JsonParser p, DeserializationContext ctx)
            throws IOException {
        return LocalDate.parse(p.getText(), DateTimeFormatter.ofPattern("dd/MM/yyyy"));
    }
}

// Uso
public class Utente {
    @JsonDeserialize(using = LocalDateDeserializer.class)
    private LocalDate dataNascita;
}
```

Serializer/deserializer custom sono necessari quando il formato JSON non corrisponde ai default di Jackson. Alternativa piu compatta: `@JsonFormat(pattern = "...")`.

## Mixin (decorare classi senza modificarle)

```java
// Non puoi modificare la classe esterna
public class ExternalDto {
    public String name;
    public String secret;
}

// Mixin: annotazioni applicate esternamente
public abstract class ExternalDtoMixin {
    @JsonIgnore
    abstract String getSecret();
}

// Applica il mixin
mapper.addMixIn(ExternalDto.class, ExternalDtoMixin.class);
```

I Mixin permettono di aggiungere annotazioni Jackson a classi che non puoi modificare (librerie esterne, classi generate). Utile per escludere campi o rinominare proprieta.

## Spring Boot: configurazione avanzata

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilder jacksonBuilder() {
        return new Jackson2ObjectMapperBuilder()
            .indentOutput(true)
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .modules(new JavaTimeModule(), new Hibernate5Module())
            .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);
    }
}
```

`Jackson2ObjectMapperBuilder` e il modo Spring Boot di configurare il `ObjectMapper` globale. `Hibernate5Module` evita problemi con proxy Hibernate (lazy loading, ghost objects).

## Errori comuni

- **`InvalidDefinitionException: No serializer found`**: la classe non ha getter pubblici o campi visibili. Aggiungi `@Data` o rendi i campi accessibili.
- **`UnrecognizedPropertyException`**: il JSON ha campi extra non mappati. Disabilita `FAIL_ON_UNKNOWN_PROPERTIES`.
- **Date serializzate come array**: `WRITE_DATES_AS_TIMESTAMPS` attivo. Disabilita e registra `JavaTimeModule`.
- **StackOverflow con JPA**: relazioni bidirezionali causano loop infinito. Usa `@JsonIgnoreProperties` o `@JsonManagedReference`/`@JsonBackReference`.
- **Cicli con `@Data` di Lombok**: `toString()` su entita JPA con relazioni lazy carica tutto. Usa `@ToString.Exclude`.
- **Generics cancellati a runtime**: usa `TypeReference` per mantenere l'informazione dei tipi generici.

## Best Practices & Conventions

- In Spring Boot, configura Jackson in `application.properties` per settaggi semplici, `Jackson2ObjectMapperBuilder` per esigenze avanzate.
- Usa **DTO specifici** per API, non esporre entita JPA direttamente (evita loop e problemi lazy).
- Usa `@JsonInclude(NON_NULL)` per ridurre dimensione delle risposte.
- Disabilita `FAIL_ON_UNKNOWN_PROPERTIES` per API pubbliche (il client puo inviare campi extra).
- Per relazioni JPA, usa `@JsonIgnoreProperties` su campi circolari e `Hibernate5Module`.
- Per date, preferisci ISO-8601 (`yyyy-MM-dd'T'HH:mm:ss`) — standard universale.
