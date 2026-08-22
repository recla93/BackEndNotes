---
topic: "Formattazione e Lint — Python"
tags: [python, lint, formatting, black, ruff, mypy, pep8]
nav_prev: "[[pytest.md]]"
nav_next: "[[Poetry e UV.md]]"
---
## PEP 8 — lo standard di stile Python

**PEP 8** (Python Enhancement Proposal 8, 2001) è il documento che definisce come scrivere codice Python "bello" e coerente. Non è obbligatorio dal compilatore (Python non controlla lo stile), ma è **seguito dalla comunità**.

### Principali regole PEP 8

| Regola | Cosa dice | Esempio ✅ | Esempio ❌ |
|---|---|---|---|
| **Indentazione** | 4 spazi, mai tab | `····x = 5` | `→x = 5` |
| **Lunghezza riga** | max 79 caratteri | riga < 79 | riga > 79 |
| **Righe vuote** | 2 tra funzioni top-level, 1 tra metodi | `def f():...\n\ndef g():` | `def f():...\ndef g():` |
| **Spazi operatori** | un spazio attorno a `=`, `+`, etc. | `x = 5 + 3` | `x=5+3` |
| **Naming** | `nome_var`, `NomeClasse`, `COSTANTE` | `nome_utente` | `nomeUtente` |
| **Import** | uno per riga, ordinati | `import os\nimport sys` | `import os, sys` |
| **Comparazione a None** | usa `is`, non `==` | `x is None` | `x == None` |
| **Tipo intero come bool** | non farlo | `if x == 0:` | `if x:` (ambiguo) |

### Perché seguire PEP 8?
- **Leggibilità**: tutto il codice Python ha lo stesso aspetto
- **Collaborazione**: nessuna guerra di stile tra sviluppatori
- **Review**: focus sulla logica, non sulla formattazione
- **Strumenti automatici**: Ruff, Black, flake8 la applicano da soli

## Ruff e Black — formattatori automatici

Il punto di PEP 8 è che **non devi pensarci tu** — gli strumenti lo fanno.

### Black: il formattatore "implacabile"
Black (2018) formatta il codice automaticamente senza opzioni (quasi):
```bash
pip install black
black src/           # formatta TUTTI i file in src/
black --check src/   # solo verifica (utile in CI)
```
Black è **opinionated**: non puoi configurare niente (tranne lunghezza riga). Vantaggio: fine delle discussioni su stile. Svantaggio: se non ti piace lo stile di Black, pazienza.

### Ruff: linter + formatter tutto-in-uno
Ruff (2022, scritto in Rust) è **10-100x più veloce** degli strumenti Python equivalenti. Rimpiazza:
- **flake8** (linter)
- **isort** (ordine import)
- **pyflakes** (errori logici)
- **pylint** (code quality)
- **black** (formattazione, con `ruff format`)

```bash
pip install ruff
ruff check src/           # trova errori di stile/logica
ruff check --fix src/     # auto-correggi
ruff format src/          # formatta (come black)
```

### Perché Ruff ha vinto
1. **Velocità**: lint di un intero progetto in millisecondi
2. **Unico tool**: lint + format + import sorting = un solo strumento
3. **Configurabile**: a differenza di Black, puoi personalizzare regole
4. **pyproject.toml**: configurazione integrata nel progetto

### In pratica
Per un progetto nuovo: **Ruff per tutto** (lint + format + isort). Aggiungi **mypy** per type checking. Configura **pre-commit** per eseguirli automaticamente a ogni commit.

## Configurazioni

Black in `pyproject.toml`:

```toml
[tool.black]
line-length = 88
target-version = ["py312"]
```

Ruff in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]

[tool.ruff.format]
quote-style = "double"
```

MyPy in `pyproject.toml`:

```toml
[tool.mypy]
strict = true
ignore_missing_imports = true
```

## Pre-commit hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.3.0
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.9.0
    hooks:
      - id: mypy
```

```bash
pip install pre-commit
pre-commit install            # attiva hooks
pre-commit run --all-files    # esegui manualmente
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| Ruff non trova nessun file | Path sbagliato o pattern glob errato | Usa `ruff check src/` con percorso esplicito |
| MyPy segnala falsi positivi su librerie | Libreria senza type stubs | Aggiungi `ignore_missing_imports = true` in `pyproject.toml` |
| Ruff e Black in conflitto | Entrambi configurati per formattare | Usa solo Ruff (`ruff format`) che è compatibile con sé stesso |
| Pre-commit blocca commit per warning non critici | Hook troppo severi | Usa `args: ["--fix"]` per auto-correct, o rilassa le regole |
| `F401` (imported but unused) in `__init__.py` | Import per re-export | Aggiungi `# noqa: F401` o configura `__all__` |

## Tool consigliati (setup minimo)

- **Ruff** per lint + format (rimpiazza black + flake8 + isort)
- **MyPy** per type checking
- **Pre-commit** per automazione git hooks
- VS Code: impostare `"ruff.enable": true` e `"python.analysis.typeCheckingMode": "strict"`
