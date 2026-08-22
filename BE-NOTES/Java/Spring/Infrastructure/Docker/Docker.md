---
topic: "Docker — containerizzazione dell'applicazione"
parent: "[[BE-NOTES/Java/Spring/Spring|Spring]]"
nav_next: "[[../MapStruct/Mapping Entity-DTO.md]]"
---


Docker containerizza l'applicazione e le sue dipendenze (database) in ambienti isolati e riproducibili. In [[TaskMngr]], Docker viene usato principalmente per avviare **PostgreSQL** in sviluppo tramite Docker Compose, evitando di installare PostgreSQL manualmente.

## Quando usare Docker

- **Database in sviluppo** — PostgreSQL, MySQL, Redis in container (zero installazione locale)
- **Ambiente riproducibile** — "Sul mio PC funziona" non esiste più: tutti usano la stessa immagine
- **CI/CD** — test in container identici alla produzione
- **Microservizi** — ogni servizio in un container separato

## Docker Compose per PostgreSQL (TaskMngr)

```yaml
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
docker compose up -d

docker compose logs -f postgres

docker compose down

docker compose down -v

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

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Porta già in uso | `Error starting userland proxy: listen tcp4 0.0.0.0:5432: bind: address already in use` | Un'altra istanza PostgreSQL o Docker è già in esecuzione sulla stessa porta | Ferma l'altro container (`docker compose down`) o cambia la porta host (`5433:5432`) |
| Volume non persistente | Dati persi dopo `docker compose down` | Usare bind mount o dimenticare named volume | Usa `volumes: - postgres_data:/var/lib/postgresql/data` con named volume |
| Container non raggiungibile | Spring Boot lancia `Connection refused` | Container non ancora avviato o nome servizio errato | Usa `docker compose up -d --wait` o `depends_on` con `condition: service_healthy` |
| Permessi volume | `Permission denied` su `/var/lib/postgresql/data` | Il container PostgreSQL è eseguito da utente diverso | Usa volume named (gestisce permessi automaticamente) o imposta `POSTGRES_USER` esplicitamente |
| `.env` non caricato | Variabili d'ambiente mancanti nel container | Docker Compose non importa automaticamente il `.env` | Docker Compose carica `.env` se presente nella stessa directory del compose file; verifica il path |
| `docker compose down -v` accidentale | Dati del database persi definitivamente | Il flag `-v` rimuove i volumi named | Usa `docker compose down` senza `-v` per fermare senza perdere dati |
| Container name conflict | `Conflict. The container name "/taskmngr-postgres" is already in use` | Nome container già usato da un container fermo | Usa `docker compose down` o `docker rm taskmngr-postgres` prima di riavviare |
