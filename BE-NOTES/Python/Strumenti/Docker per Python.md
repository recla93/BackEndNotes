---
topic: "Docker per Python"
tags: [python, docker, container, deploy, devops]
nav_prev: "[[Poetry e UV.md]]"
---
Riferimento ufficiale: [docs.docker.com/language/python](https://docs.docker.com/language/python/)

Docker containerizza l'app Python con tutte le dipendenze, garantendo ambienti identici in sviluppo, CI e produzione.

Vedi anche: [[BE-NOTES/Python/FastAPI/Testing e Deploy|FastAPI Deploy]], [[BE-NOTES/Python/Django/Auth e Deploy|Django Deploy]], [[BE-NOTES/Python/Strumenti/Poetry e UV|UV/Poetry]] per layer caching.

`FROM python:3.12-slim` usa l'immagine ufficiale Python basata su Debian slim (~120MB vs ~900MB full). Il layer delle dipendenze (`COPY requirements.txt + RUN pip install`) è separato dal layer del codice — Docker cachea i layer; se `requirements.txt` non cambia, il layer delle dipendenze non viene ricostruito. `WORKDIR /app` imposta la directory di lavoro e crea il path se non esiste. `CMD` in formato exec (JSON array) — mai in formato shell per evitare PID 1 issues.

```dockerfile
# Python ufficiale immagine slim
FROM python:3.12-slim

# Directory di lavoro
WORKDIR /app

# Installa dipendenze (prima del codice per cache layer)
COPY requirements.txt .
RUN pip install --no-cache-dir -v -r requirements.txt

# Copia il resto del codice
COPY . .

# Espone porta
EXPOSE 8000

# Comando di avvio
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Dockerfile multi-stage

Il multi-stage separa build e runtime. `AS builder` è il primo stage: installa le dipendenze con `--user` (le mette in `/root/.local`). Il secondo stage copia solo i binari delle dipendenze (`COPY --from=builder`), senza layer inutili — l'immagine finale è più piccola e non contiene il layer di build. `ENV PATH=...` serve per trovare gli eseguibili installati con `--user`.

```dockerfile
# Build stage
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Final stage
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## docker-compose per app + DB

`depends_on` con `condition: service_healthy` aspetta che il DB sia pronto prima di avviare l'app — evita race condition all'avvio. `volumes: .:/app` monta il codice locale nel container (hot reload in sviluppo). `healthcheck` esegue `pg_isready` ogni 5s per verificare che PostgreSQL sia pronto. `postgres_data` è un volume named che persiste i dati oltre la vita del container. `SECRET_KEY=${SECRET_KEY}` legge dal file `.env` locale (o dall'ambiente).

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - SECRET_KEY=${SECRET_KEY}
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - .:/app  # solo sviluppo

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s

volumes:
  postgres_data:
```

## .dockerignore

```
__pycache__/
*.pyc
.venv/
.git/
.gitignore
.env
*.md
.DS_Store
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `pip install` lento a ogni build | `COPY . .` prima di `RUN pip install` — nessun layer caching | Copia `requirements.txt` PRIMA del codice |
| Container muore subito | `CMD` in formato shell (`uvicorn ...`) invece di exec | Usa `CMD ["uvicorn", "main:app", ...]` |
| `pg_isready` non trovato | `healthcheck` eseguito dentro container PostgreSQL | Usa `CMD-SHELL` — il comando esiste nell'immagine PostgreSQL |
| `SECRET_KEY` vuota in produzione | Variabile d'ambiente non passata al container | Usa `env_file: .env` o Docker secrets |
| Immagine >1GB | `FROM python:3.12` (full) invece di `-slim` | Usa `python:3.12-slim` o multi-stage |
| `operation not permitted` | Esecuzione come root | Aggiungi `USER appuser` e `RUN adduser appuser` |

## Best practice

- Usare `python:3.12-slim` (non alpine — mancano librerie)
- Non eseguire come root: `USER appuser`
- `pip install --no-cache-dir` per immagini più piccole
- Ordinare i layer: prima requirements, poi codice (caching)
- Usare variabili d'ambiente per configurazione
- `--healthcheck` per servizi dipendenti
- Mai hardcodare secret nelle immagini
