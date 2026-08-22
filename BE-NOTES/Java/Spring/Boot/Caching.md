---
topic: "Caching — Spring Boot"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

Spring Boot fornisce caching dichiarativo via annotazioni. `@Cacheable` memorizza il risultato di un metodo, `@CacheEvict` invalida la cache, `@CachePut` aggiorna la cache senza saltare l'esecuzione.

Il caching riduce il carico su DB e servizi esterni per dati che cambiano raramente. Spring supporta diversi cache manager: in-memory (ConcurrentHashMap), Redis, Caffeine, EhCache, JCache (JSR-107).

## Abilitazione e cache manager

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        // In-memory (semplice, default)
        return new ConcurrentMapCacheManager("utenti", "articoli");

        // Caffeine (performante, con TTL e limiti)
        CaffeineCacheManager caffeine = new CaffeineCacheManager("utenti");
        caffeine.setCaffeine(Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(1000));
        return caffeine;
    }
}
```

`@EnableCaching` su una classe `@Configuration` attiva il caching. `ConcurrentMapCacheManager` e il default (senza scadenza). Caffeine offre controllo su TTL, dimensione massima e statistiche.

## @Cacheable

```java
@Service
public class UserService {

    @Cacheable(value = "utenti", key = "#id")
    public UserDto getUser(Long id) {
        // chiamata al DB (eseguita solo se cache miss)
        return repository.findById(id).orElseThrow();
    }

    // Key complesso
    @Cacheable(value = "utenti", key = "#email + ':' + #type")
    public UserDto findByEmailAndType(String email, String type) {
        return repository.findByEmailAndType(email, type);
    }

    // Cache condizionale
    @Cacheable(value = "utenti", condition = "#id > 1000")
    public UserDto getUserConditional(Long id) {
        return repository.findById(id).orElseThrow();
    }
}
```

`@Cacheable` controlla prima la cache: se la chiave esiste, restituisce il valore cachato senza eseguire il metodo. `key` usa SpEL per definire la chiave. `condition` abilita il caching solo se la condizione e vera.

## @CacheEvict e @CachePut

```java
@Service
public class UserService {

    // Invalida la cache dopo aggiornamento
    @CacheEvict(value = "utenti", key = "#id")
    public void updateUser(Long id, UserUpdate dto) {
        repository.update(id, dto);
    }

    // Invalida tutta la cache "utenti"
    @CacheEvict(value = "utenti", allEntries = true)
    public void refreshAllUsers() {
        // operazione di refresh
    }

    // Aggiorna la cache (esegue sempre il metodo)
    @CachePut(value = "utenti", key = "#result.id")
    public UserDto createUser(CreateUser request) {
        User saved = repository.save(new User(request));
        return mapper.toDto(saved);
    }
}
```

`@CacheEvict` rimuove dalla cache (prima o dopo l'esecuzione del metodo via `beforeInvocation`). `allEntries = true` svuota tutta la cache. `@CachePut` esegue sempre il metodo e aggiorna la cache con il risultato.

## Caching su piu cache

```java
@Cacheable(cacheNames = {"utenti", "utenti-sintetici"}, key = "#id")
public UserDto getUser(Long id) {
    return repository.findById(id).orElseThrow();
}

// Cache multipla
@Caching(
    evict = {
        @CacheEvict(value = "utenti", key = "#id"),
        @CacheEvict(value = "utenti-by-email", key = "#email")
    },
    put = @CachePut(value = "utenti", key = "#id")
)
public UserDto update(Long id, String email, UserUpdate dto) {
    return repository.update(id, dto);
}
```

`cacheNames` permette di scrivere su piu cache contemporaneamente. `@Caching` raggruppa operazioni multiple (evict, put, cacheable) in una singola annotazione.

## Redis come cache

```properties
# application.properties
spring.cache.type=redis
spring.data.redis.host=localhost
spring.data.redis.port=6379
# TTL globale
spring.cache.redis.time-to-live=10m
# Permetti null (default: false)
spring.cache.redis.cache-null-values=true
```

```java
// Nessuna configurazione aggiuntiva necessaria
@Service
public class ProductService {

    @Cacheable(value = "prodotti", key = "#id")
    public ProductDto getProduct(Long id) {
        return repository.findById(id).orElseThrow();
    }
}
```

Con `spring.cache.type=redis`, Spring usa Redis come backend. Nessuna modifica al codice: le annotazioni rimangono identiche. TTL configurabile per cache o globale.

## Cache con Redis avanzato

```java
@Configuration
public class RedisCacheConfig {

    @Bean
    public RedisCacheManagerBuilderCustomizer cacheCustomizer() {
        return builder -> builder
            .withCacheConfiguration("utenti",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(10))
                    .disableCachingNullValues())
            .withCacheConfiguration("prodotti",
                RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofHours(1))
                    .prefixCacheNameWith("prod::"));
    }
}
```

`RedisCacheManagerBuilderCustomizer` permette configurazioni diverse per cache diverse: TTL, prefisso, serializzazione, compressione.

## Statistiche cache (Caffeine)

```java
@Configuration
public class CacheMonitor {

    @Bean
    public CacheMetricsRegistrar cacheMetrics(CacheManager cacheManager,
            MeterRegistry meterRegistry) {
        return new CacheMetricsRegistrar(cacheManager, meterRegistry);
    }
}

// Con Actuator
// Esporta metriche delle cache via /actuator/metrics
// cache.gets, cache.puts, cache.evictions, cache.hit.ratio
```

Usa Actuator + Micrometer per monitorare hit ratio, miss, eviction, dimensione. Fondamentale per verificare se la cache sta effettivamente funzionando.

## Cache a livello di query (Hibernate)

```java
// application.properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

Hibernate offre cache di secondo livello per entita e query. Configurabile con JCache (JSR-107). Utile per entita lette frequentemente e raramente modificate (catalogo prodotti, stati, configurazioni).

## Errori comuni

- **Cache senza TTL**: memoria piena. Imposta sempre TTL (`expireAfterWrite`).
- **Cache key collision**: chiavi troppo generiche. Usa namespace (`utenti:123`, non `123`).
- **@Cacheable su metodo privato**: non funziona (proxy Spring). I metodi devono essere `public`.
- **Stale data**: dati aggiornati nel DB ma non in cache. Usa `@CacheEvict` dopo update/delete.
- **Cache di collezioni grandi**: se la cache cresce senza limiti, OutOfMemoryError. Imposta `maximumSize`.
- **Serializzazione con Redis**: gli oggetti cachati devono essere serializzabili (implementare `Serializable` o usare JSON).

## Best Practices & Conventions

- Usa `@Cacheable` su metodi che accedono a DB o servizi esterni con dati che cambiano raramente.
- Imposta sempre **TTL** (`expireAfterWrite`). Mai cache infinita.
- Invalida la cache con `@CacheEvict` dopo create/update/delete.
- Usa chiavi descrittive: `"utenti:" + #user.id + ":" + #user.email`.
- Monitora hit ratio in produzione (Actuator + Prometheus).
- Per cache condivisa tra istanze, usa Redis.
- Per cache locale (singola istanza), Caffeine e piu veloce di ConcurrentHashMap.
