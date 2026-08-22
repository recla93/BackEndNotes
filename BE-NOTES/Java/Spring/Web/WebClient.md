---
topic: "WebClient — HTTP Client reattivo"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
nav_prev: "[[Controller e REST.md]]"
---

`WebClient` e il client HTTP moderno di Spring WebFlux. Sostituisce `RestTemplate` (dichiarato in maintenance mode). E reattivo (non bloccante) per default ma supporta anche uso sincrono con `block()`.

A differenza di `RestTemplate` (bloccante, thread per richiesta), `WebClient` e non bloccante: un thread gestisce piu richieste simultaneamente. Ideale per microservizi e chiamate API concorrenti.

## Configurazione base

```java
import org.springframework.web.reactive.function.client.WebClient;

// Base
WebClient client = WebClient.create("https://api.example.com");

// Con configurazione
WebClient client = WebClient.builder()
    .baseUrl("https://api.example.com")
    .defaultHeader("Authorization", "Bearer " + token)
    .defaultHeader("Content-Type", "application/json")
    .build();
```

`WebClient` e thread-safe e va configurato una volta come bean Spring. Non creare istanze per ogni richiesta. Inietta il bean configurato nei service.

## Uso sincrono (block)

```java
// GET
WebClient.ResponseSpec response = client.get()
    .uri("/users/{id}", 1L)
    .retrieve();

UserDto user = response.bodyToMono(UserDto.class).block();  // bloccante

// POST
UserDto nuovo = client.post()
    .uri("/users")
    .bodyValue(new CreateUserRequest("Alice", "alice@test.com"))
    .retrieve()
    .bodyToMono(UserDto.class)
    .block();
```

`.block()` rende la chiamata sincrona. Usalo solo in contesti bloccanti (controller MVC, service sincroni). In contesti reattivi, non chiamare mai `block()`.

## Uso reattivo (Mono/Flux)

```java
import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

// Singolo risultato
Mono<UserDto> userMono = client.get()
    .uri("/users/{id}", 1L)
    .retrieve()
    .bodyToMono(UserDto.class);

// Lista di risultati
Flux<UserDto> usersFlux = client.get()
    .uri("/users")
    .retrieve()
    .bodyToFlux(UserDto.class);

// Pipeline reattiva
usersFlux
    .filter(user -> user.getEta() > 18)
    .map(UserDto::getNome)
    .subscribe(System.out::println);
```

`Mono<T>` per 0-1 risultato, `Flux<T>` per 0-N risultati. La pipeline non viene eseguita finche non ci si subscribe. Non chiamare mai `block()` in contesto reattivo (blocca l'event loop).

## Gestione errori

```java
Mono<UserDto> result = client.get()
    .uri("/users/{id}", id)
    .retrieve()
    .onStatus(HttpStatusCode::is4xxClientError, response ->
        Mono.error(new UserNotFoundException("Utente non trovato: " + id))
    )
    .onStatus(HttpStatusCode::is5xxServerError, response ->
        response.bodyToMono(ErrorResponse.class)
            .flatMap(err -> Mono.error(new ExternalServiceException(err.message())))
    )
    .bodyToMono(UserDto.class)
    .onErrorResume(TimeoutException.class, ex -> {
        log.warn("Timeout chiamata utente {}", id);
        return Mono.just(new UserDto("default"));
    })
    .timeout(Duration.ofSeconds(5));
```

`onStatus()` intercetta codici HTTP specifici. `onErrorResume()` fornisce un fallback. `timeout()` annulla la richiesta se supera il limite. Combina gli operatori reattivi per una gestione errori granulare.

## Request personalizzata

```java
Mono<String> result = client.get()
    .uri(uriBuilder -> uriBuilder
        .path("/api/users")
        .queryParam("page", 0)
        .queryParam("size", 20)
        .queryParam("sort", "nome,asc")
        .build())
    .headers(headers -> headers.setBearerAuth(token))
    .cookie("session", sessionId)
    .ifNoneModified(Instant.now())
    .accept(MediaType.APPLICATION_JSON)
    .retrieve()
    .bodyToMono(String.class);
```

`uriBuilder` costruisce URI con query params. `headers()` modifica gli header. `ifNoneModified()` aggiunge `If-Modified-Since`. `cookie()` aggiunge cookie. `accept()` imposta `Accept`.

## Exchange (basso livello)

```java
Mono<ClientResponse> exchange = client.get()
    .uri("/users/{id}", id)
    .exchange();  // deprecated in WebClient 1.1+

// Alternativa: retrieve con accesso alla response
Mono<ResponseEntity<UserDto>> response = client.get()
    .uri("/users/{id}", id)
    .retrieve()
    .toEntity(UserDto.class);
```

`retrieve()` e preferibile per la maggior parte dei casi (gestisce automaticamente status e body). `toEntity()` fornisce accesso a header e status code insieme al body. `exchange()` e deprecato da Spring WebFlux 1.1.

## Bean Spring

```java
@Configuration
public class WebClientConfig {

    @Bean
    public WebClient webClient(WebClient.Builder builder) {
        return builder
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
}
```

Inietta `WebClient.Builder` (fornito da Spring Boot) e personalizzalo. WebClient e thread-safe: un singolo bean e sufficiente per tutta l'applicazione.

## WebClient vs RestTemplate

| Caratteristica | RestTemplate | WebClient |
|---------------|-------------|-----------|
| Natura | Bloccante | Non bloccante / bloccante |
| Thread per richiesta | Si | No |
| Reattivo | No | Si (Mono/Flux) |
| Status | **Deprecato** | Attivo |
| Performance | Buona | Migliore (alta concorrenza) |
| API | Imperativa | Funzionale |
| Error handling | try-catch | onStatus/onErrorResume |
| Quando usare | Legacy | Nuovo sviluppo |

## Errori comuni

- **`block()` in contesto reattivo**: blocca l'event loop reattivo. Mai chiamare `block()` in un controller WebFlux.
- **WebClient creato ad ogni richiesta**: istanziare WebClient e costoso. Crea un bean singleton.
- **Timeout non configurato**: richieste che rimangono in attesa per minuti. Usa `.timeout(Duration.ofSeconds(5))`.
- **Dimenticare `onStatus()`**: errori 4xx/5xx lanciano `WebClientResponseException` generica. Gestiscile specificamente.
- **Mischiare retrieve() ed exchange()**: preferisci `retrieve()`. Usa `toEntity()` se servono header.
- **WebClient senza Spring Boot**: devi aggiungere `spring-boot-starter-webflux`.

## Best Practices & Conventions

- Usa **WebClient** per tutto il nuovo codice. `RestTemplate` e deprecato.
- Configura **WebClient come bean singleton**.
- Imposta sempre **timeout** con `.timeout()`.
- Gestisci errori HTTP con `onStatus()`.
- Per microservizi, usa `WebClient` reattivo senza `block()`.
- Per controller MVC (Tomcat), puoi usare `.block()` — ma considera WebFlux per nuova architettura.
- Inietta `WebClient.Builder` e personalizzalo, non creare `WebClient` con `new`.
