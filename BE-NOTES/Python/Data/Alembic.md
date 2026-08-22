---
topic: "Alembic — Migrazioni Database"
tags: [python, alembic, migrations, database]
nav_prev: "[[SQLAlchemy ORM.md]]"
nav_next: "[[Pandas.md]]"
---
Riferimento ufficiale: [alembic.sqlalchemy.org](https://alembic.sqlalchemy.org/)

Alembic è lo strumento di migrazione ufficiale per SQLAlchemy (come Flyway per Java o Django `makemigrations`). Genera script Python che modificano lo schema DB in modo versionato e riproducibile.

### A differenza di Django migrations:
- Alembic richiede configurazione manuale (`env.py`)
- Supporta autogenerate solo se `target_metadata` è configurato
- Più flessibile per schema complessi e database multipli

Vedi anche: [[BE-NOTES/Python/Data/SQLAlchemy ORM|ORM]] (i modelli da migrare), [[BE-NOTES/Python/Data/Database Connection|Connection]] (DB URL in alembic.ini).

`alembic init alembic` crea la struttura di directory: una cartella `alembic/` con `env.py`, `script.py.mako`, `versions/`, e un file `alembic.ini` nella root del progetto.

```bash
pip install alembic
alembic init alembic
```

Crea la struttura:
```
progetto/
├── alembic/
│   ├── versions/     # migrazioni
│   ├── env.py        # configurazione
│   └── script.py.mako
└── alembic.ini       # connessione DB
```

## Configurazione

`target_metadata = Base.metadata` è il collegamento tra modelli e migrazioni: senza questo, `--autogenerate` non può rilevare le modifiche ai modelli. `engine_from_config` legge la stringa di connessione da `alembic.ini`.

```python
# alembic/env.py
from sqlalchemy import engine_from_config
from alembic import context
from app.models import Base  # i tuoi modelli

target_metadata = Base.metadata  # molto importante!
```

## Comandi

`--autogenerate` confronta `Base.metadata` (stato attuale dei modelli) con lo schema reale del DB e genera la migrazione automaticamente. `upgrade head` applica tutte le migrazioni pendenti fino all'ultima. `downgrade -1` fa rollback di una versione. `alembic current` mostra la revisione corrente del DB; `alembic history` mostra l'intera cronologia.

```bash
# Creare migrazione (autogenerata dai modelli)
alembic revision --autogenerate -m "descrizione"

# Applicare migrazioni (porta DB all'ultima versione)
alembic upgrade head

# Rollback (downgrade)
alembic downgrade -1  # una versione indietro
alembic downgrade abc123  # version specific

# Vedere stato
alembic current        # versione attuale
alembic history        # cronologia
```

## Migrazione manuale

`upgrade()` contiene le operazioni da applicare (DDL). `downgrade()` deve essere l'inverso esatto — se `upgrade` crea una tabella, `downgrade` la droppa; se aggiunge una colonna, la rimuove. `op.*` sono operazioni DDL portabili: funzionano su PostgreSQL, MySQL, SQLite, ecc. Lo script è generato da `alembic revision` (senza `--autogenerate`) o scrivibile a mano.

```python
"""empty message

Revision ID: abc123
Revises: def456
Create Date: 2024-01-01 10:00:00
"""
from alembic import op
import sqlalchemy as sa

revision = "abc123"
down_revision = "def456"

def upgrade():
    op.create_table(
        "nuova_tabella",
        sa.Column("id", sa.Integer, primary_key=True),
        sa.Column("nome", sa.String(100), nullable=False),
    )
    op.add_column("utenti", sa.Column("telefono", sa.String(20)))

def downgrade():
    op.drop_column("utenti", "telefono")
    op.drop_table("nuova_tabella")
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `Target database is not up to date` | DB in una revisione diversa da `head` | Esegui `alembic upgrade head` |
| Autogenerate non rileva modifiche | `target_metadata` non configurato o modelli non importati | Verifica `Base.metadata` in `env.py` |
| `FAILED: No revisions in this dir` | Cartella `versions/` vuota | Crea prima revisione con `alembic revision --autogenerate` |
| Downgrade fallisce con FK violation | `downgrade()` non inverte `upgrade()` nell'ordine corretto | Inverti esattamente l'ordine delle operazioni |
| `alembic check` non trova nulla | Modifica non ancora salvata nel modello | Verifica che il modello sia importato in `env.py` |

## Best practice

- `--autogenerate` per >90% delle migrazioni
- **Sempre** verificare la migrazione prima di applicarla
- Versionare le migrazioni (commit)
- Mai modificare migrazioni già applicate — crearne di nuove
- Testare `upgrade` e `downgrade` in sviluppo
- `alembic check` per vedere se ci sono modifiche non migrate
