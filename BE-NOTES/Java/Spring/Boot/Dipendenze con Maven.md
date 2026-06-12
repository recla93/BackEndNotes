---
topic: "Dipendenze con Maven"
parent: "[[BE-NOTES/Java/Spring/Boot/Core Concepts|Core Concepts]]"
---

# Dipendenze con Maven

Maven gestisce il ciclo di vita del progetto: compilazione, test, packaging, deploy. [[TaskMngr]] usa **Maven Wrapper** (`mvnw`), che garantisce che tutti usino la stessa versione di Maven senza installarla manualmente.

## Parent BOM — versioni gestite centralmente

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>4.0.6</version>
</parent>
```

Il **parent POM** di Spring Boot è un BOM (Bill of Materials): dichiara le versioni di TUTTE le dipendenze compatibili. Quando aggiungi uno starter, non devi specificare la versione — il parent la fornisce.

## Dipendenze chiave di TaskMngr

| Dipendenza | Scopo | Alternativa |
|---|---|---|
| `spring-boot-starter-data-jpa` | JPA + Hibernate + HikariCP | MyBatis, JDBC template |
| `postgresql` | Driver PostgreSQL | H2 (dev), MySQL |
| `spring-boot-starter-web` | REST API + Tomcat embedded | WebFlux (reattivo) |
| `spring-boot-starter-security` | Filter chain + auth base | Nessuna (obbligatorio per sicurezza) |
| `spring-boot-starter-oauth2-client` | Login Google/GitHub | Auth0, Keycloak |
| `spring-boot-starter-validation` | Bean Validation (Hibernate Validator) | Manuale |
| `springdoc-openapi-starter-webmvc-ui` | OpenAPI + Swagger UI | SpringFox (deprecato) |
| `mapstruct` + `mapstruct-processor` | Mapping compile-time Entity↔DTO | Manuale, ModelMapper (runtime) |
| `lombok` | Riduzione boilerplate | Record Java (parziale) |
| `me.paulschwarz:spring-dotenv` | Carica `.env` all'avvio | Spring Cloud Config |

## Maven Wrapper — build riproducibile

```bash
mvnw.cmd clean compile       # compila
mvnw.cmd clean test          # esegue test
mvnw.cmd clean package -DskipTests  # genera jar
mvnw.cmd spring-boot:run     # avvia l'applicazione
```

Il wrapper è un piccolo script + file `.mvn/wrapper/maven-wrapper.properties`. Garantisce che tutti (dev, CI, prod) usino Maven 3.9+ senza installazione globale.

## Attenzione a MapStruct + Lombok

MapStruct e Lombok processano entrambi le annotazioni in compile-time. L'ordine conta: **Lombok deve girare prima di MapStruct** (Lombok genera getter/setter, MapStruct li usa). In Maven:

```xml
<annotationProcessorPaths>
    <path>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </path>
    <path>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-processor</artifactId>
    </path>
</annotationProcessorPaths>
```

Se vedi `Can't map property "X"`, probabilmente l'ordine è sbagliato.
