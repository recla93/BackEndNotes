---
topic: "Application Properties"
parent: "[[BE-NOTES/Java/Spring/Boot/Core Concepts|Core Concepts]]"
nav_prev: "[[Autoconfiguration e Starters.md]]"
nav_next: "[[Dipendenze con Maven.md]]"
---


Spring Boot externalizza la configurazione: stesso jar, comportamenti diversi in base all'ambiente. Supporta file YAML/properties, variabili d'ambiente, argomenti CLI e file `.env`.

## Quando usare cosa

| Fonte | Esempio | Quando usarla |
|---|---|---|
| `application.yml` | Config principale | Valori di default, dev environment |
| `application-{profile}.yml` | `application-prod.yml` | Config specifica per ambiente |
| `.env` | Segreti locali | Sviluppo locale, **mai committato** |
| Variabili d'ambiente | `JWT_SECRET=...` | Produzione, CI/CD, Docker |
| Argomenti CLI | `--server.port=8081` | Override temporanei, test |

## Ordinamento di priorità (dal meno al più prioritario)

1. **File default** (`application.yml`) — base comune
2. **Profilo specifico** (`application-dev.yml`) — sovrascrive solo ciò che serve
3. **Variabili d'ambiente** — per secret in produzione
4. **Argomenti CLI** — `--jwt.secret=abc` override tutto

Una property definita in una fonte più prioritaria sovrascrive la stessa property nelle fonti meno prioritarie.

## Esempio concreto (TaskMngr)

**application.yml** (base, committato):
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/taskmanager
    username: postgres
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

**application-prod.yml** (override, committato):
```yaml
spring:
  jpa:
    show-sql: false        # disabilita SQL log in produzione
    hibernate:
      ddl-auto: validate   # non modificare lo schema in produzione
```

**.env** (NON committato, nel .gitignore):
```
JWT_SECRET=supersecretkey2024
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=yyy
SPRING_DATASOURCE_PASSWORD=password_prod
```

**Classe `@ConfigurationProperties` tipizzata**:
```java
@ConfigurationProperties(prefix = "app.jwt")
@Validated
public class JwtProperties {
    @NotBlank
    private String secret;        // from app.jwt.secret
    private long expirationMs = 86400000;  // default 24h

    // getter/setter obbligatori
}
```

Evita `@Value("${app.jwt.secret}")` sparsi nel codice — raggruppa le property in classi coese.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Secret hardcodato in application.yml | Credenziali in VCS, leak in repo pubblico | La password/chiave è scritta direttamente nel file YAML | Usa `${VARIABILE_AMBIENTE}` e `.env` locale |
| Profilo non attivato | Le property del profilo non vengono caricate | `application-prod.yml` ignorato se non attivi il profilo | Usa `--spring.profiles.active=prod` o `SPRING_PROFILES_ACTIVE=prod` |
| `@ConfigurationProperties` senza getter/setter | Bean non creato, errore all'avvio | Spring richiede getter e setter per bindare le property | Aggiungi getter/setter (o usa `record` con Spring Boot 3+) |
| Property riferita ma mancante | `IllegalArgumentException` o valore null | `${PROPERTY}` non definita in nessuna fonte | Usa `${PROPERTY:valore_default}` per dare un default |
| Nomi property Java vs YAML inconsistenti | `@Value("${app.jwt.secret}")` non trova la property | `app.jwt.secret` vs `app.jwt-secret` in YAML | I separatori YAML ('.' e '-') sono entrambi validi ma usali consistentemente |
| `.env` non caricato in esecuzione | Le variabili d'ambiente locali non vengono lette | Spring Boot non carica `.env` automaticamente | Aggiungi `@PropertySource(".env")` o usa il plugin `spring-boot-maven-plugin` con `.env` |

## Best practice

- **Mai hardcodare segreti** — sempre in `.env` (dev) o variabili d'ambiente (prod)
- **Raggruppa per prefisso** — `app.jwt.*`, `app.oauth2.*`, `app.upload.*`
- **Usa `@Validated`** sulle classi `@ConfigurationProperties` — fallisci subito se manca una property obbligatoria
- **Non committare `.env`** — aggiungilo subito al `.gitignore`
- **Prefisso stabile** — cambiare `prefix` rompe tutti i bindings
