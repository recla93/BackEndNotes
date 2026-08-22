---
topic: "Spring Doc — documentazione API con OpenAPI"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
nav_prev: "[[../MapStruct/MapStruct.md]]"
---


Spring Doc genera automaticamente documentazione **OpenAPI 3.0** per le API Spring Boot, esponendo una UI Swagger interattiva per testare gli endpoint. Sostituisce SpringFox (non più mantenuto).

## Quando usare Spring Doc

- **API REST pubbliche** — ogni API dovrebbe avere documentazione aggiornata automaticamente
- **Frontend-backend separation** — il frontend può esplorare gli endpoint senza chiedere al backend
- **Test manuali** — Swagger UI permette di chiamare endpoint con autenticazione, body, parametri
- **Code generation** — dallo spec OpenAPI si possono generare client Angular, React, mobile
- **Team onboarding** — nuovo sviluppatore capisce subito l'API

## Configurazione

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
</dependency>
```

Aggiungi la dipendenza → hai Swagger UI su `/swagger-ui.html` e lo spec JSON su `/v3/api-docs`. Nessuna configurazione extra.

## Personalizzazione

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("TaskMngr API")
                .version("1.0")
                .description("API per la gestione di task con autenticazione JWT e OAuth2"))
            .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")));
    }
}
```

## Annotazioni utili

```java
@Operation(summary = "Crea un nuovo task", description = "Solo utenti autenticati")
@ApiResponse(responseCode = "201", description = "Task creato")
@ApiResponse(responseCode = "400", description = "Validazione fallita")
@PostMapping
public ResponseEntity<ApiResponse<TaskDto>> create(
        @RequestBody @Valid TaskCreateRequest request) { ... }
```

Spring Doc documenta automaticamente:
- **Parametri** — da `@RequestParam`, `@PathVariable`, `@RequestBody`
- **Response** — dal tipo di ritorno del controller
- **Status code** — da `@ResponseStatus` o `ResponseEntity`
- **Validazione** — da `@NotBlank`, `@Size`, etc.
- **Pageable** — da `@ParameterObject` (documenta `?page=`, `?size=`, `?sort=`)

## In TaskMngr

- `/swagger-ui.html` → UI interattiva
- `/v3/api-docs` → spec OpenAPI JSON (usabile per generare client)
- Bearer JWT configurabile tramite il bottone "Authorize" in Swagger UI
- Endpoint pubblici (auth, oauth2) accessibili senza token
- Endpoint protetti → clicca "Authorize", incolla il JWT, testa le chiamate

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Endpoint non visibile in Swagger UI | Il controller/method non appare nella UI | Manca `@Operation` o il metodo non è mappato con `@GetMapping` etc. | Verifica che il controller sia annotato con `@RestController` e i metodi con mapping HTTP |
| Security scheme non configurato | Bottone "Authorize" non funziona o non presente | Manca la configurazione `SecurityScheme` nel bean `OpenAPI` | Aggiungi `components.addSecuritySchemes("bearerAuth", ...)` in `OpenApiConfig` |
| 401/403 non documentati | Swagger mostra solo 200 OK | Manca `@ApiResponse` per error code | Aggiungi `@ApiResponse(responseCode = "401", description = "Unauthorized")` |
| Spring Doc non si carica | 404 su `/swagger-ui.html` o `/v3/api-docs` | Dipendenza mancante o conflitto con Spring Security | Aggiungi `springdoc-openapi-starter-webmvc-ui` e permetti accesso in SecurityConfig |
| Parametri `@RequestParam` non documentati | Manca il campo nella UI | Il parametro non ha `@Parameter(description = "...")` | Aggiungi `@Parameter(description = "Filtro per nome")` o descrivi il parametro |
| `Pageable` non espanso | Swagger mostra un solo campo "pageable" invece di page/size/sort | Manca `@ParameterObject` o Spring Doc non riconosce il parametro | Usa `@ParameterObject` sul parametro `Pageable` o configura esplicitamente |
| API documentata ma endpoint protetto non testabile | 401 su ogni chiamata da Swagger | JWT non configurato nel bottone Authorize | Configura Bearer token in OpenAPI config e incolla un JWT valido |
