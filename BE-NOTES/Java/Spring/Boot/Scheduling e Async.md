---
topic: "Scheduling e Async in Spring Boot"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

Spring Boot supporta esecuzione programmata (`@Scheduled`) e asincrona (`@Async`) tramite annotazioni. `@Scheduled` per task ciclici (cron, fixed rate). `@Async` per esecuzione non bloccante con thread pool separato.

Senza abilitazione esplicita, le annotazioni vengono ignorate. `@EnableScheduling` e `@EnableAsync` attivano la funzionalita.

## Abilitazione e thread pool

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.EnableAsync;
import java.util.concurrent.Executor;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import org.springframework.context.annotation.Bean;

@Configuration
@EnableScheduling
@EnableAsync
public class AsyncConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

`@EnableScheduling` attiva `@Scheduled`. `@EnableAsync` attiva `@Async`. Senza `@Bean` Executor personalizzato, Spring usa `SimpleAsyncTaskExecutor` (crea un thread per ogni task, senza pool). Per produzione, configura sempre pool thread.

## @Scheduled — schedulazione

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledTasks {

    // Ogni 5 secondi
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        // Eseguito ogni 5 secondi, indipendentemente dalla durata
        log.info("Ora: {}", LocalTime.now());
    }

    // 5 secondi dopo il completamento del precedente
    @Scheduled(fixedDelay = 5000)
    public void processBatch() {
        // Attende 5s dalla fine dell'esecuzione precedente
        processItems();
    }

    // Con initial delay
    @Scheduled(fixedRate = 5000, initialDelay = 10000)
    public void delayedTask() {
        // Primo avvio dopo 10s, poi ogni 5s
        syncExternalData();
    }

    // Espressione cron
    @Scheduled(cron = "0 0 2 * * ?")
    public void nightlyCleanup() {
        // Ogni notte alle 02:00
        deleteOldRecords();
    }
}
```

`fixedRate` = intervallo tra inizio e inizio successivo (sovrapposizione possibile). `fixedDelay` = attesa dopo la fine del precedente (mai sovrapposizione). `cron` = espressione cron standard (6 campi: second, minute, hour, day, month, day-of-week).

## @Async — esecuzione asincrona

```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class NotificationService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject, String body) {
        try {
            emailClient.send(to, subject, body);
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            return CompletableFuture.failedFuture(e);
        }
    }

    @Async("customExecutor")
    public CompletableFuture<String> fetchData(Long id) {
        UserData data = externalApi.fetch(id);
        return CompletableFuture.completedFuture(data.getName());
    }
}
```

`@Async` esegue il metodo in un thread separato (dal pool configurato). Il chiamante non attende il completamento. Ritorna `void` (fire-and-forget) o `Future`/`CompletableFuture` (per monitoraggio). `@Async("customExecutor")` usa un pool specifico per nome bean.

## @Async con risultato

```java
@Service
public class PriceService {

    @Async
    public CompletableFuture<BigDecimal> getPriceFromProviderA(Long productId) {
        BigDecimal price = apiA.getPrice(productId);
        return CompletableFuture.completedFuture(price);
    }

    @Async
    public CompletableFuture<BigDecimal> getPriceFromProviderB(Long productId) {
        BigDecimal price = apiB.getPrice(productId);
        return CompletableFuture.completedFuture(price);
    }
}

@RestController
public class PriceController {

    @Autowired
    private PriceService priceService;

    @GetMapping("/products/{id}/best-price")
    public Mono<BigDecimal> getBestPrice(@PathVariable Long id) throws Exception {
        CompletableFuture<BigDecimal> a = priceService.getPriceFromProviderA(id);
        CompletableFuture<BigDecimal> b = priceService.getPriceFromProviderB(id);

        // Esecuzione parallela, prendi il piu veloce
        CompletableFuture<BigDecimal> fastest = CompletableFuture
            .anyOf(a, b)
            .thenApply(r -> (BigDecimal) r);

        return Mono.fromFuture(fastest);
    }
}
```

`CompletableFuture` permette composizione di operazioni asincrone: `allOf` (aspetta tutti), `anyOf` (aspetta il primo), `thenApply` (trasforma il risultato). Wrapping in `Mono`/`Flux` per integrazione reattiva.

## Schedulazione condizionale

```java
@Component
public class ConditionalScheduler {

    private final AtomicBoolean enabled = new AtomicBoolean(true);

    public void pause() { enabled.set(false); }
    public void resume() { enabled.set(true); }

    @Scheduled(fixedRate = 10000)
    public void conditionalTask() {
        if (!enabled.get()) {
            log.info("Task disabilitato, skip");
            return;
        }
        doWork();
    }
}
```

Spring non supporta `@Scheduled(condition = ...)`. Usa un flag `AtomicBoolean` per abilitare/disabilitare dinamicamente. Per controllo centralizzato, usa `TaskScheduler` programmaticamente.

## Programmatic scheduling

```java
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.support.CronTrigger;

@Service
public class DynamicScheduler {

    @Autowired
    private TaskScheduler taskScheduler;

    private ScheduledFuture<?> scheduledTask;

    public void startTask(Runnable task, String cronExpression) {
        if (scheduledTask != null && !scheduledTask.isCancelled()) {
            scheduledTask.cancel(false);
        }
        scheduledTask = taskScheduler.schedule(task, new CronTrigger(cronExpression));
    }

    public void stopTask() {
        if (scheduledTask != null) {
            scheduledTask.cancel(false);
        }
    }
}
```

`TaskScheduler` permette schedulazione programmatica (senza annotazioni). Utile per task che cambiano a runtime (es. cron configurabile da DB). `CronTrigger` accetta espressioni cron dinamiche.

## Gestione errori

```java
@Component
public class ScheduledTasks {

    @Scheduled(fixedRate = 5000)
    public void taskWithErrorHandling() {
        try {
            doRiskyWork();
        } catch (Exception e) {
            log.error("Task fallito, riprovero al prossimo ciclo", e);
            // Non rilancia: il prossimo ciclo riprova
            // Se rilancia, il task non viene piu eseguito
        }
    }
}

// Global error handler per @Async
@Configuration
public class AsyncExceptionConfig implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(Throwable ex, Method method, Object... params) {
        log.error("Async method {} ha fallito: {}", method.getName(), ex.getMessage(), ex);
    }
}
```

`@Scheduled` non rilancia eccezioni per default (logga e continua). Se rilancia, il task non viene piu eseguito. `@Async` senza `CompletableFuture` non notifica errori al chiamante. Implementa `AsyncUncaughtExceptionHandler` per gestire eccezioni da metodi `@Async` void.

## Errori comuni

- **Dimenticare `@EnableScheduling`**: `@Scheduled` viene ignorato senza l'annotazione di abilitazione.
- **Thread pool non configurato**: `SimpleAsyncTaskExecutor` crea infiniti thread. Configura sempre pool.
- **`@Async` su metodo privato**: ignorato (proxy Spring). Deve essere `public`.
- **Chiamata diretta a metodo `@Async`**: bypassa il proxy. Deve passare dal bean iniettato.
- **Schedulazione senza gestione errori**: task che falliscono silenziosamente. Logga sempre le eccezioni.
- **Cron su datetime sbagliato**: espressioni cron complesse possono essere errate. Testa con cron-utils.
- **FixedRate sovrapposto**: se il task dura piu del rate, i thread si accumulano. Usa `fixedDelay` o pool adeguato.

## Best Practices & Conventions

- Usa `@Scheduled` per task ciclici prevedibili (pulizia, sincronizzazione, reporting).
- Usa `@Async` per operazioni lunghe che non devono bloccare il chiamante (email, notifiche, processamento).
- Configura sempre `ThreadPoolTaskExecutor` con limiti (core, max, queue).
- Usa `CompletableFuture` per tracciare il completamento dei metodi `@Async`.
- Gestisci errori dentro `@Scheduled` (logga e continua, non rilancia).
- Per task dinamici (configurabili da DB), usa `TaskScheduler` programmatico.
- Usa espressioni cron in file di properties: `@Scheduled(cron = "${my.cron.expression}")`.
