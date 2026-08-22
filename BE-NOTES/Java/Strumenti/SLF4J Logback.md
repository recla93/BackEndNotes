---
topic: "SLF4J e Logback — Logging"
parent: "[[BE-NOTES/Java/Java|Java]]"
---

SLF4J (Simple Logging Facade for Java) è un'astrazione che permette di cambiare implementazione di logging (Logback, Log4j2, java.util.logging) senza modificare il codice. Logback è l'implementazione predefinita in Spring Boot.

A differenza di `System.out.println()`, SLF4J supporta: livelli (TRACE, DEBUG, INFO, WARN, ERROR), output su file/console con rotazione, formattazione configurabile, e valutazione lazy dei messaggi.

## Logger base

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserService {
    private static final Logger log = LoggerFactory.getLogger(UserService.class);

    public void processaUtente(Long id) {
        log.info("Elaborazione utente: {}", id);
        log.debug("Dettagli: {}", getDettagli(id));
        log.warn("Utente {} non trovato", id);
        log.error("Errore DB", exception);  // logga eccezione con stack trace
    }
}
```

`{}` e il placeholder SLF4J. Non usare concatenazione di stringhe: `log.info("msg " + variabile)` valuta sempre la stringa. SLF4J valuta il placeholder solo se il livello e attivo — risparmio CPU.

## Con Lombok

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class UserService {
    public void metodo() {
        log.info("Logger generato da Lombok");
    }
}
```

`@Slf4j` genera automaticamente `private static final Logger log = LoggerFactory.getLogger(ThisClass.class)`. Alternativa: `@Log4j2` per Log4j2.

## logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Console: colori per sviluppo -->
    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <!-- File con rotazione giornaliera (produzione) -->
    <springProfile name="prod">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>logs/app.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>logs/app-%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>30</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>

        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>
</configuration>
```

`<springProfile>` attiva configurazioni diverse per profilo Spring (`dev`, `prod`). `%highlight` aggiunge colori in console. `TimeBasedRollingPolicy` crea un nuovo file ogni giorno e tiene 30 backup.

## application.properties per logging

```properties
# Livello globale
logging.level.root=INFO

# Livello specifico per package
logging.level.com.example=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.springframework.security=TRACE

# Output su file
logging.file.name=logs/app.log
logging.logback.rollingpolicy.max-history=30
logging.logback.rollingpolicy.max-file-size=10MB

# Pattern
logging.pattern.console=%d{HH:mm:ss} %-5level %logger{36} - %msg%n
```

Spring Boot permette di configurare logging base senza file XML. Utile per settaggi rapidi. Per configurazioni complesse (file separati per profilo, appender custom), usa `logback-spring.xml`.

## MDC (Mapped Diagnostic Context)

```java
import org.slf4j.MDC;

// Nel filtro/interceptor
MDC.put("userId", user.getId().toString());
MDC.put("requestId", UUID.randomUUID().toString());

// In tutto il thread corrente, i log includeranno userId e requestId
log.info("Operazione completata");

// Pulisci sempre
MDC.clear();
```

```xml
<!-- Pattern con MDC -->
<pattern>%d{HH:mm:ss} [%X{requestId}] %-5level %logger{36} - %msg%n</pattern>
```

MDC associa valori al thread corrente. Ogni log emesso da quel thread include automaticamente i valori MDC. Essenziale per tracciare richieste in applicazioni multi-thread.

## Performance e best practice

```java
// ✅ BUONO: valutazione lazy
log.debug("Risultato: {}", calcoloCostoso());

// ✅ MEGLIO: guard per operazioni costose (se il livello DEBUG non e attivo)
if (log.isDebugEnabled()) {
    log.debug("Risultato: {}", calcoloMoltoCostoso());
}

// ❌ MALE: valutazione a prescindere
log.debug("Risultato: " + calcoloCostoso());
```

SLF4J valuta il placeholder lazy (chiama `toString()` solo se il livello e attivo). Per operazioni molto costose, usa `isDebugEnabled()` come guard. In produzione, DEBUG e TRACE sono solitamente disabilitati.

## Livelli e quando usarli

| Livello | Quando usarlo |
|---------|--------------|
| TRACE | Dettaglio fine solo per debug approfondito |
| DEBUG | Info utili in sviluppo, non in produzione |
| INFO | Eventi importanti di business: richieste, operazioni completate |
| WARN | Situazioni anomale ma non critiche: deprecation, fallback |
| ERROR | Errori recuperabili: eccezioni gestite, transazioni fallite |

## Errori comuni

- **`System.out.println()` in produzione**: nessun controllo livelli, nessuna rotazione, difficile da filtrare.
- **Loggare dati sensibili**: password, token, dati personali. Usa filtri o maschera i campi.
- **Concatenazione con `+`**: valuta sempre la stringa. Usa placeholder `{}`.
- **Dimenticare MDC.clear()**: valori MDC residui contaminano richieste successive. Pulisci in `finally` o filtro.
- **Log eccessivo (log storm)**: milioni di log al minuto affogano il sistema e i costi di storage. Livello INFO per eventi significativi, non per ogni iterazione.
- **Stack trace senza contesto**: `log.error("Errore")` senza passare l'eccezione perde lo stack. Usa `log.error("msg", exception)` - l'eccezione va come secondo argomento.

## Best Practices & Conventions

- Usa sempre **SLF4J** come facade, non l'implementazione diretta.
- In Spring Boot, Logback e preconfigurato. Non serve dipendenza esplicita.
- Usa **`@Slf4j`** di Lombok per meno boilerplate.
- Usa **MDC** per correlare log tra servizi (requestId, userId, traceId).
- Configura **file rotanti** in produzione (mai un singolo file).
- Non loggare in loop critici per performance.
- Usa livelli appropriati: INFO per operazioni business, DEBUG per dettagli implementativi.
