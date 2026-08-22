---
topic: "Python — linguaggio di programmazione"
tags: [python, backend, scripting, fastapi, django, data]
---


Linguaggio interpretato, tipizzazione dinamica forte, multi-paradigma (OOP, funzionale, procedurale). Usato per backend (FastAPI, Django), scripting, automazione, data processing, bot.

## Aree

- [[BE-NOTES/Python/Core Concepts/Variabili e Tipi|Base]] — fondamenti: tipi, controllo, funzioni, collezioni, file, errori, moduli
- [[BE-NOTES/Python/OOP/Classi e Oggetti|OOP]] — classi, ereditarietà, property, magic methods, decoratori
- [[BE-NOTES/Python/Funzionale/Concetti Base|Funzionale]] — lambda, map/filter/reduce, itertools, functools
- [[BE-NOTES/Python/Tecnologie/Async e AsyncIO|Tecnologie]] — async, generatori, thread/processi, context manager, type hints
- [[BE-NOTES/Python/FastAPI/Setup e Primi Passi|FastAPI]] — API REST moderne, Pydantic, SQLAlchemy, auth, testing
- [[BE-NOTES/Python/Django/Setup e Struttura|Django]] — framework full-stack, ORM, admin, DRF, deploy
- [[BE-NOTES/Python/Data/Database Connection|Data]] — SQLAlchemy Core/ORM, Pandas, Alembic migrazioni
- [[BE-NOTES/Python/Strumenti/pytest|Strumenti]] — pytest, linting, formattazione, poetry/uv, Docker

## Convenzioni generali

### PEP 8 — lo standard di stile Python
**PEP 8** (Python Enhancement Proposal 8) è il documento ufficiale che definisce come scrivere codice Python leggibile e coerente. Non è obbligatorio, ma è seguito dalla comunità. Cosa dice:
- **Indentazione**: 4 spazi (mai tab)
- **Lunghezza riga**: max 79 caratteri (codice), 72 (commenti)
- **Naming**: `nome_variabile` (snake_case), `NomeClasse` (PascalCase), `COSTANTE` (UPPER_CASE)
- **Spazi**: un operatore = un spazio ai lati (`x = 5`, non `x=5`)
- **Import**: una riga per import, ordinati standard → third-party → locali

Oggi **Ruff** e **Black** applicano PEP 8 automaticamente ([[BE-NOTES/Python/Strumenti/Formattazione e Lint|Formattazione e Lint]]).

### Altre convenzioni
- **Type hints** — annotare tipi (`nome: str`) migliora leggibilità e permette a mypy di trovare bug ([[BE-NOTES/Python/Tecnologie/Type Hints|dettaglio]])
- **Docstring** — triple quote `"""..."""` subito dopo `def` o `class` per documentare cosa fa
- **Named arguments** — nei parametri non ovvi, specifica il nome: `crea_utente(nome="Mario", eta=25)` invece di `crea_utente("Mario", 25)`
- **EAFP** — "Easier to Ask for Forgiveness than Permission": provi l'operazione, gestisci l'errore se fallisce ([[BE-NOTES/Python/Core Concepts/Errori e Eccezioni|dettaglio]])
- **Composizione > ereditarietà** — costruisci oggetti combinando componenti invece di ereditare comportamenti ([[BE-NOTES/Python/OOP/Classi e Oggetti|dettaglio]])
- **Pathlib > os.path** — `pathlib.Path()` è più leggibile e cross-platform di `os.path.join()` ([[BE-NOTES/Python/Core Concepts/File e IO|dettaglio]])
- **F-strings** — l'interpolazione `f"Ciao {nome}"` è più veloce e leggibile di `%` e `.format()` ([[BE-NOTES/Python/Core Concepts/Variabili e Tipi|dettaglio]])
