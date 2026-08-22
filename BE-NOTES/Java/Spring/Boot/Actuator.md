---
topic: "Actuator — Monitoring e Management"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

Spring Boot Actuator fornisce endpoint HTTP per monitoring, health, metriche, e diagnostica dell'applicazione in produzione. Essenziale per osservabilita in ambienti containerizzati e orchestrati.

Actuator espone endpoint su `/actuator` per default. Richiede il modulo `spring-boot-starter-actuator` e puo essere esposto su porta separata per sicurezza.

## Setup base

```xml
<!-- Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# application.properties
management.endpoints.web.exposure.include=health,info,metrics
management.endpoints.web.base-path=/manage
management.server.port=8081  # Porta separata per sicurezza
```

`management.endpoints.web.exposure.include` abilita gli endpoint via HTTP. Per default solo `health` e `info` sono esposti. `management.server.port` usa una porta diversa da quella dell'applicazione (sicurezza).

## Endpoint principali

```properties
# Esposizione endpoint
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans

# Health dettagliato
management.endpoint.health.show-details=always
# oppure when-authorized (default)
management.endpoint.health.show-components=always

# Info endpoint (custom)
info.app.name=@project.name@
info.app.version=@project.version@
info.app.java-version=21
```

`*` espone tutti gli endpoint (attento in produzione). `env` e `beans` contengono informazioni sensibili — escludili in produzione. `show-details=always` mostra health dettagliato di tutti i componenti.

## Endpoint disponibili

| Endpoint | Descrizione | Sensibile |
|----------|-------------|-----------|
| `/health` | Stato applicazione + componenti (DB, Redis, Kafka, ...) | No |
| `/info` | Metadati (versione, nome, contatti) | No |
| `/metrics` | Metriche JVM, HTTP, cache, DB pool, ... | No |
| `/env` | Properties caricate | Si |
| `/beans` | Tutti i bean Spring | Si |
| `/configprops` | @ConfigurationProperties | Si |
| `/loggers` | Logging level per package (GET/POST) | Moderato |
| `/heapdump` | Heap dump (file) | Si |
| `/threaddump` | Thread dump | Si |
| `/mappings` | Endpoint mapping (@RequestMapping) | Moderato |

## Health personalizzato

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class ExternalApiHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean apiUp = checkExternalApi();

        if (apiUp) {
            return Health.up()
                .withDetail("url", "https://api.example.com")
                .withDetail("latency", measureLatency())
                .build();
        }

        return Health.down()
            .withDetail("url", "https://api.example.com")
            .withDetail("error", "API non raggiungibile")
            .build();
    }
}
```

`HealthIndicator` personalizzato controlla servizi esterni. `Health.up()` / `Health.down()` determinano lo stato aggregato. `withDetail()` aggiunge meta-informazioni utili per il debugging.

## Info endpoint custom

```java
import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class CustomInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder
            .withDetail("deployedAt", Instant.now())
            .withDetail("git.commit",
                Map.of("hash", gitProperties.getCommitId(),
                       "branch", gitProperties.getBranch()))
            .withDetail("contacts",
                Map.of("team", "backend@company.com",
                       "onCall", "+39 123 456 7890"));
    }
}
```

`InfoContributor` aggiunge metadati all'endpoint `/info`. Utile per tracciare deployment, commit, branch, contatti del team. Combina con `git-commit-id-plugin` per informazioni Git automatiche.

## Metriche con Micrometer

```properties
# Esposizione Prometheus
management.endpoints.web.exposure.include=prometheus,health,metrics

# Metriche JVM
management.metrics.tags.application=${spring.application.name}

# Metriche HTTP
management.metrics.web.server.request.autotime.enabled=true
```

```java
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Counter;

@Service
public class OrderService {

    private final Counter orderCounter;

    public OrderService(MeterRegistry registry) {
        this.orderCounter = Counter.builder("orders.created")
            .tag("currency", "EUR")
            .description("Numero ordini creati")
            .register(registry);
    }

    public Order createOrder(CreateOrder request) {
        Order order = repository.save(new Order(request));
        orderCounter.increment();
        return order;
    }
}
```

Micrometer e il facade metriche di Spring Boot. `MeterRegistry` crea counter, timer, gauge. Supporta backend: Prometheus, Datadog, Graphite, New Relic. Per Prometheus, aggiungi `micrometer-registry-prometheus`.

## Prometheus integration

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
# prometheus.yml (cattura ogni 15s)
scrape_configs:
  - job_name: 'myapp'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8081']
```

Con `micrometer-registry-prometheus`, Actuator espone `/actuator/prometheus` nel formato Prometheus. Prometheus cattura le metriche a intervalli regolari. Grafana visualizza dashboard.

## Loggers endpoint

```bash
# Leggi livello log per un package
GET /actuator/loggers/com.example

# Cambia livello a runtime (POST)
POST /actuator/loggers/com.example
{
  "configuredLevel": "DEBUG"
}
```

L'endpoint `/loggers` permette di leggere e modificare i livelli di log a runtime. Utilissimo per debugging temporaneo in produzione senza riavviare. Il cambiamento non persiste al riavvio.

## Env endpoint

```json
// GET /actuator/env
{
  "propertySources": [
    {
      "name": "systemEnvironment",
      "properties": {
        "PATH": { "value": "..." }
      }
    },
    {
      "name": "application.properties",
      "properties": {
        "server.port": { "value": "8080" }
      }
    }
  ]
}
```

`/env` mostra tutte le properties caricate (incluse variabili d'ambiente). **Sensibile**: password, token, secreti sono visibili. Non esporre in produzione senza autenticazione.

## Sicurezza Actuator

```java
import static org.springframework.boot.actuate.autoconfigure.security.servlet
    .EndpointRequest.toAnyEndpoint;

@Configuration
public class ActuatorSecurity {

    @Bean
    public SecurityFilterChain actuatorFilterChain(HttpSecurity http) throws Exception {
        http.securityMatcher("/actuator/**")
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health")
                    .permitAll()
                .requestMatchers("/actuator/info")
                    .permitAll()
                .requestMatchers(toAnyEndpoint())
                    .hasRole("ADMIN")
            )
            .httpBasic(withDefaults());
        return http.build();
    }
}
```

Endpoint sensibili protetti con Spring Security. `health` e `info` pubblici (necessari per orchestrazione). Altri endpoint richiedono ADMIN. Oppure usa porta separata (`management.server.port`) e firewall.

## Errori comuni

- **Endpoint non esposto**: `management.endpoints.web.exposure.include` non configurato.
- **Secreti in `/env`**: password e token visibili. Proteggi con Spring Security.
- **Health UP ma app non funzionale**: health indicator superficiali. Verifica dipendenze reali (DB, coda, API esterna).
- **Metriche senza tag**: difficili da aggregare. Aggiungi tag (application, instance, region).
- **Heap dump endpoint senza limite**: `/heapdump` puo generare file da GB. Limita accesso.
- **Prometheus senza registry**: `micrometer-registry-prometheus` mancante. Solo metriche di base.
- **Loggers endpoint modificato in produzione**: dimenticato di ripristinare il livello. Usa con cautela e documenta.

## Best Practices & Conventions

- Esponi solo `health` e `info` pubblicamente. Proteggi gli altri endpoint.
- Usa `management.server.port` per isolare Actuator su porta separata.
- Implementa `HealthIndicator` per ogni dipendenza esterna (DB, Redis, API, Kafka).
- Usa Micrometer + Prometheus + Grafana per metriche e alerting.
- Configura `management.metrics.tags.application` per filtrare metriche per servizio.
- Esponi `/prometheus` per monitoring in produzione.
- Usa endpoint `/loggers` per debugging temporaneo (solo se autenticato).
- Aggiungi `InfoContributor` con commit, branch, contatti team.
