---
topic: "Profiles in Spring Boot"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

I profili Spring permettono configurazioni diverse per ambienti (dev, test, staging, prod). Si attivano una o piu configurazioni specifiche senza modificare il codice.

Spring carica `application.properties` sempre, poi carica `application-{profile}.properties` per ogni profilo attivo. Le properties del profilo sovrascrivono quelle generiche.

## File di properties per profilo

```properties
# application.properties (condivise)
app.name=MyService
spring.datasource.url=jdbc:h2:mem:testdb

# application-dev.properties
server.port=8080
spring.datasource.url=jdbc:h2:file:./data/dev
logging.level.com.example=DEBUG

# application-prod.properties
server.port=80
spring.datasource.url=jdbc:postgresql://prod-db:5432/app
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
logging.level.com.example=WARN
```

Nominazione standard: `application-{profile}.properties` (o `.yml`). Spring le carica in base al profilo attivo. I placeholder `${}` risolvono variabili d'ambiente o system properties.

## Attivazione profilo

```bash
# Via VM option (IDE o java -jar)
java -jar myapp.jar -Dspring.profiles.active=prod

# Via environment variable (produzione)
export SPRING_PROFILES_ACTIVE=prod

# In application.properties
spring.profiles.active=dev
```

Ordine di precedenza (dal piu prioritario): command line > env variable > application.properties. In produzione, usa environment variable (`SPRING_PROFILES_ACTIVE`) per non hardcodare il profilo.

## Profili multipli attivi

```properties
# application.properties
spring.profiles.active=dev,italy
```

Profili multipli separati da virgola. Utile per combinazioni: `dev + cloud`, `prod + eu`. L'ordine conta: l'ultimo sovrascrive in caso di conflitto.

## Bean condizionali per profilo

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://prod-db:5432/app");
        ds.setUsername(System.getenv("DB_USERNAME"));
        ds.setPassword(System.getenv("DB_PASSWORD"));
        return ds;
    }

    @Bean
    @Profile("!test")  // Tutto tranne test
    public DataSource defaultDataSource() {
        return new H2DataSource();
    }
}
```

`@Profile("dev")` carica il bean solo se `dev` e attivo. `@Profile("!test")` esclude il profilo `test`. Espressioni complesse: `"dev | staging"` (OR), `"dev & cloud"` (AND), `"!prod"` (NOT).

## ApplicationRunner per profilo

```java
@Component
@Profile("dev")
public class DataInitializer implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        log.info("Caricamento dati di test...");
        // Crea dati fittizi per sviluppo
        userRepository.save(new User("admin", "admin@dev.com"));
    }
}
```

`ApplicationRunner` + `@Profile` esegue codice all'avvio solo per un ambiente specifico. Utile per seed data (dev), health check (prod), metriche iniziali (staging).

## YAML multi-documento

```yaml
# application.yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:default

---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 80
  datasource:
    url: jdbc:postgresql://prod-db:5432/app
---
spring:
  config:
    activate:
      on-profile: dev
logging:
  level:
    com.example: DEBUG
```

YAML multi-documento (separato da `---`) definisce profili nello stesso file. `spring.config.activate.on-profile` sostituisce `spring.profiles` (deprecato). Preferisci file separati per profili grandi.

## @ConfigurationProperties per profilo

```java
@ConfigurationProperties(prefix = "app.config")
@Validated
public class AppConfig {
    @NotBlank
    private String environment;
    private List<String> allowedOrigins;
    // getters e setters
}

// application-dev.properties
app.config.environment=development
app.config.allowedOrigins=http://localhost:3000,http://localhost:8080

// application-prod.properties
app.config.environment=production
app.config.allowedOrigins=https://myapp.com
```

`@ConfigurationProperties` si lega alle properties in base al profilo attivo. Ogni profilo puo fornire valori diversi. `@Validated` con `@NotBlank` fallisce all'avvio se manca.

## Profilo di default

```java
@Configuration
@Profile("default")
public class DefaultConfig {
    // Caricato solo se nessun profilo e attivo
}

// Oppure in application.properties:
spring.profiles.default=dev  # default se nessun profilo specificato
```

Se nessun profilo e attivo, Spring usa `default`. Puoi configurare il profilo di default via `spring.profiles.default`. Separa configurazioni obbligatorie (default) da specifiche per ambiente.

## Priorita delle properties

```
1. @TestPropertySource (test)
2. Command line (--server.port=8081)
3. SPRING_APPLICATION_JSON (env var)
4. ServletConfig init params
5. JNDI
6. System properties (-Dkey=value)
7. OS environment variables
8. application-{profile}.properties
9. application.properties
10. @PropertySource
```

Le properties da fonti con priorita maggiore sovrascrivono quelle a priorita minore. I profili si trovano a meta classifica. Variabili d'ambiente e command line hanno priorita piu alta dei profili.

## Errori comuni

- **Profilo non attivo**: bean `@Profile("prod")` non caricato in dev. Verifica profilo attivo nei log di avvio.
- **Typos nei nomi profilo**: `develop` vs `dev`. Usa costanti o enum per i nomi profilo.
- **Properties non caricate**: file `application-prod.properties` mancante. Il profilo si attiva ma le properties specifiche non esistono.
- **YAML multi-documento malformato**: spazi, trattini, indentazione. YAML e sensibile all'indentazione.
- **Secreti nei file properties**: mai password/token hardcodati in `application-prod.properties`. Usa variabili d'ambiente.
- **Profili attivi da piu fonti**: command line + env + application.properties. Controlla l'ordine di precedenza.
- **Bean definito in due profili senza default**: se `@Profile("dev")` e `@Profile("prod")` definiscono stesso bean, e ok se uno e attivo. Se nessuno e attivo, manca il bean.

## Best Practices & Conventions

- Usa profili **solo per configurazione**, non per logica di business.
- Non hardcodare `spring.profiles.active` in `application.properties`. Usa env variable in produzione.
- Nomi profilo: `dev`, `test`, `staging`, `prod`.
- Separa secreti in variabili d'ambiente (`${DB_PASSWORD}`), non nei file properties.
- Documenta i profili disponibili nel README.
- Usa gruppi di profili (Spring Boot 2.4+) per combinazioni comuni.
- Per test, usa `@ActiveProfiles("test")` + file `application-test.properties`.
