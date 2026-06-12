---
topic: "Docker — containerizzazione dell'applicazione"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
---

# Docker

Docker containerizza l'applicazione e le sue dipendenze (database) in ambienti isolati e riproducibili. In [[TaskMngr]], Docker viene usato principalmente per avviare **PostgreSQL** in sviluppo tramite Docker Compose, evitando di installare PostgreSQL manualmente.

## Quando usare Docker

- **Database in sviluppo** — PostgreSQL, MySQL, Redis in container (zero installazione locale)
- **Ambiente riproducibile** — "Sul mio PC funziona" non esiste più: tutti usano la stessa immagine
- **CI/CD** — test in container identici alla produzione
- **Microservizi** — ogni servizio in un container separato

## Docker Compose per PostgreSQL (TaskMngr)

```yaml
# docker-compose.yml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine          # immagine leggera (alpine = ~150MB)
    container_name: taskmngr-postgres
    environment:
      POSTGRES_DB: taskmanager
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}   # default: postgres
    ports:
      - "5432:5432"                     # host:container — mappa la porta standard
    volumes:
      - postgres_data:/var/lib/postgresql/data  # dati persistenti oltre il restart
    restart: unless-stopped

volumes:
  postgres_data:                        # named volume, non un bind mount
```

## Comandi utili

```bash
# Avvia PostgreSQL in background
docker compose up -d

# Vedi i log del database
docker compose logs -f postgres

# Ferma tutto (senza eliminare i volumi)
docker compose down

# Ferma TUTTO e cancella i dati (pericoloso in produzione)
docker compose down -v

# Entra nel container per eseguire psql
docker exec -it taskmngr-postgres psql -U postgres -d taskmanager
```

## Collegamento dell'applicazione a Docker PostgreSQL

Nelle `application-dev.yml` di [[BE-NOTES/Java/Spring/Boot/Application Properties|Spring Boot]]:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/taskmanager
    username: postgres
    password: postgres
```

Il container espone PostgreSQL su `localhost:5432` — l'applicazione Spring Boot (in esecuzione sulla macchina host) si connette normalmente.

## Dockerfile per l'applicazione (produzione)

```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/taskmngr-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

In produzione, anche l'applicazione va containerizzata e orchestrata con Docker Compose o Kubernetes.

## In TaskMngr

- Docker Compose per PostgreSQL in sviluppo
- Volume named `postgres_data` per persistenza dati (non perdi dati tra un `docker compose down` e l'altro)
- `.env` per configurare password DB
- Spring Boot si connette a `localhost:5432` fuori dal container o al nome servizio dentro la rete Docker
