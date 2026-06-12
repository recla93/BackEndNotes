---
topic: "Autoconfiguration e Starters"
parent: "[[BE-NOTES/Java/Spring/Boot/Core Concepts|Core Concepts]]"
---

# Autoconfiguration e Starters

Spring Boot elimina la configurazione manuale che caratterizzava Spring classico (XML, `@Enable*` espliciti). Lo fa con due meccanismi: **starters** (dipendenze preassemblate) e **autoconfiguration** (bean creati automaticamente in base alle librerie presenti).

## Starters — dipendenze pronte all'uso

Uno starter è un raggruppamento di dipendenze coerenti. Invece di dichiarare 5 jar separati, ne dichiari uno:

| Starter | Cosa include | Quando serve |
|---|---|---|
| `spring-boot-starter-web` | Tomcat + Spring MVC + Jackson | API REST |
| `spring-boot-starter-data-jpa` | Hibernate + HikariCP (pool) + DataSource + Spring Data JPA | Accesso a database relazionale |
| `spring-boot-starter-security` | Spring Security + filtri base | Autenticazione e autorizzazione |
| `spring-boot-starter-oauth2-client` | OAuth2 client + login sociale | Login con Google/GitHub |
| `spring-boot-starter-validation` | Hibernate Validator + Bean Validation API | Validazione DTO con `@Valid` |

Non serve aggiungere le singole librerie — lo starter le trascina tutte, con versioni compatibili tra loro (gestite dal BOM Spring Boot).

## Autoconfiguration — come Spring Boot decide cosa configura

Quando l'applicazione parte, Spring Boot scandisce le classi elencate in:
```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

Per ogni classe, valuta una serie di **condizioni**:

```java
@Configuration
@ConditionalOnClass(DataSource.class)                // Si attiva solo se DataSource è nel classpath
@ConditionalOnMissingBean(DataSource.class)           // Solo se non hai già definito un DataSource
@ConditionalOnProperty(prefix = "spring.datasource") // Solo se la property è impostata
public class DataSourceAutoConfiguration { ... }
```

Questo significa che **aggiungere una dipendenza è sufficiente** per abilitarne la configurazione: se aggiungi `spring-boot-starter-data-jpa`, Spring Boot vede `DataSource`, `EntityManager`, `JpaRepository` nel classpath e configura automaticamente datasource, dialect Hibernate, transaction manager, ecc.

## Quando creare una autoconfiguration personalizzata

In progetti complessi potresti creare una libreria condivisa (es. `common-security-starter`) che:
1. Definisce `@Configuration` condizionali in `AutoConfiguration.imports`
2. Usa `@ConditionalOnProperty` per attivare/disattivare features
3. Espone `@ConfigurationProperties` per la personalizzazione

**In TaskMngr** non serve — la configurazione è gestita direttamente nell'app.

## Disabilitare autoconfiguration indesiderate

```yaml
spring:
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

Utile quando vuoi gestire manualmente un bean che Spring Boot configurerebbe automaticamente.

## In TaskMngr

- Properties personalizzate con `@ConfigurationProperties(prefix = "app")` per: JWT secret, OAuth2 redirect URIs, limiti upload
- Profili `dev`/`prod` con file `application-dev.yml`, `application-prod.yml`
- Autoconfiguration di Spring Security + JPA + Web usate come-default
