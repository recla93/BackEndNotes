---
topic: "Poetry e UV"
tags: [python, poetry, uv, packaging, dependencies]
nav_prev: "[[Formattazione e Lint.md]]"
nav_next: "[[Docker per Python.md]]"
---
L'ecosistema Python offre diversi strumenti per gestire dipendenze e ambienti virtuali. **Poetry** è il più maturo per progetti da pubblicare (lock file + build + publish). **UV** è il più veloce per installazioni e CI (Rust-based, 10-100x più veloce di pip).

Vedi anche: [[BE-NOTES/Python/Core Concepts/Moduli e Pacchetti|Moduli e Pacchetti]] per struttura progetto, [[BE-NOTES/Python/Strumenti/Docker per Python|Docker]] per ambienti containerizzati.

`poetry new` crea la struttura progetto con `pyproject.toml`, cartella `src/` e `tests/`. `poetry add` installa la dipendenza E la aggiunge al `pyproject.toml` — unico comando (pip richiede `pip install` + modifica manuale). `poetry install` legge `poetry.lock` se esiste (installazione deterministica) o lo genera se manca. `poetry shell` attiva il virtual env creato e gestito automaticamente da Poetry. `poetry export` genera `requirements.txt` per ambienti che non supportano Poetry.

```bash
pip install poetry

# Nuovo progetto
poetry new myproject
poetry init                    # in progetto esistente

# Aggiungere dipendenze
poetry add fastapi sqlalchemy
poetry add --group dev pytest pytest-cov

# Installare da pyproject.toml
poetry install

# Eseguire script nell'ambiente
poetry run python main.py

# Attivare shell
poetry shell

# Esportare requirements.txt
poetry export -f requirements.txt --output requirements.txt

# Build e publish
poetry build
poetry publish
```

pyproject.toml (con Poetry):

```toml
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""
authors = ["Mario Rossi <mario@test.it>"]

[tool.poetry.dependencies]
python = "^3.12"
fastapi = "^0.115.0"
sqlalchemy = "^2.0"

[tool.poetry.group.dev.dependencies]
pytest = "^8.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

## UV — installer ultra-veloce (Rust)

UV è scritto in Rust e usa un resolver più efficiente di pip. `uv pip install` ha la stessa sintassi di pip ma è 10-100x più veloce — ideale per CI. `uv venv .venv` crea un virtual env standard nella directory di progetto. `uv tool install` installa strumenti globali (come `pipx` ma più veloce). UV non ha lock file né build system: è un **sostituto di pip**, non di Poetry.

```bash
pip install uv

# Equivalente pip
uv pip install fastapi
uv pip install -r requirements.txt
uv pip freeze

# Virtual env
uv venv .venv
uv pip sync requirements.txt

# Strumenti globali
uv tool install ruff
uv tool run black src/

# Vantaggi: 10-100x più veloce di pip
# Compatibile con requirements.txt e pyproject.toml
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `poetry.lock` non aggiornato | Dipendenza aggiunta ma `poetry lock` non eseguito | `poetry lock --no-update` per rigenerare lock |
| Poetry non trova Python giusto | `pyproject.toml` richiede `^3.12` ma Python system è 3.11 | Usa `pyenv` o imposta `[tool.poetry] python-path` |
| UV installa versione sbagliata | UV non ha resolver di dipendenze avanzato | Usa `uv pip install -r requirements.txt` con versioni bloccate |
| `poetry build` fallisce con moduli mancanti | `packages` non configurato in `pyproject.toml` | Aggiungi `packages = [{include = "mio_modulo"}]` |

## Confronto

| Feature | Poetry | UV | pip |
|---|---|---|---|
| Velocità | Media | Molto alta | Bassa |
| Lock file | poetry.lock | — | — |
| Build/Publish | ✅ | — | — |
| Virtual env | Auto | ✅ (uv venv) | Manuale |
| Dependency resolver | ✅ | — | ❌ |
| Compatibilità | pyproject.toml | pip-compat | requirements.txt |

## Consiglio

- **UV** per installazioni veloci e CI
- **Poetry** per progetti da pubblicare o che richiedono lock file deterministico
- Entrambi convivono tranquillamente
