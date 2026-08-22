---
topic: "Tox e Nox — Automazione Test Python"
tags: [python, testing, tox, nox, automation, ci]
---
Riferimento ufficiale Tox: [tox.wiki](https://tox.wiki/)
Riferimento ufficiale Nox: [nox.thea.codes](https://nox.thea.codes/)

Tox e Nox sono strumenti di automazione che creano ambienti virtuali isolati per eseguire test su più versioni di Python e dipendenze. Risolvono il problema "sulla mia macchina funziona" garantendo che il codice sia testato in ambienti puliti e riproducibili.

Tox usa un file `tox.ini` dichiarativo. Nox usa un `noxfile.py` programmatico (Python puro). Tox è più configurazione-dichiarativa; Nox è più flessibile e adatto a pipeline complesse.

Vedi anche:
[[BE-NOTES/Python/Strumenti/pytest|pytest]],
[[BE-NOTES/Python/Strumenti/Poetry e UV|Poetry e UV]],
[[BE-NOTES/Python/Strumenti/Coverage|Coverage]].

## Tox — configurazione dichiarativa

```ini
# tox.ini
[tox]
envlist = py39, py310, py311, lint
skip_missing_interpreters = true

[testenv]
deps =
    pytest
    pytest-cov
commands =
    pytest tests/ --cov=mio_package

[testenv:lint]
deps =
    ruff
commands =
    ruff check .
```

`tox run` crea un ambiente virtuale per ogni versione di Python, installa le dipendenze, esegue i comandi e riporta i risultati. `skip_missing_interpreters` evita errori se Python 3.9 non è installato.

## Tox — ambienti multipli

```ini
[tox]
envlist = py{39,310,311}-django{32,40}, lint

[testenv]
deps =
    django32: Django>=3.2,<4.0
    django40: Django>=4.0,<5.0
    pytest
commands = pytest

[testenv:lint]
deps = ruff
commands = ruff check .
```

Tox supporta matrici di ambienti con la sintassi delle parentesi graffe. `py39-django32` e `py39-django40` sono ambienti distinti. Ogni ambiente è completamente isolato.

## Nox — configurazione programmatica

```python
# noxfile.py
import nox

@nox.session(python=["3.9", "3.10", "3.11"])
def tests(session):
    session.install("pytest", "pytest-cov")
    session.run("pytest", "tests/", "--cov=mio_package")


@nox.session(python="3.11")
def lint(session):
    session.install("ruff")
    session.run("ruff", "check", ".")
```

In Nox ogni sessione è una funzione Python. `session.install()` e `session.run()` gestiscono l'ambiente virtuale automaticamente. Più flessibile di Tox per condizioni e logiche complesse.

## Nox — parametri e condizioni

```python
import nox

@nox.session(python="3.11")
@nox.parametrize("django", ["3.2", "4.0", "4.1"])
def tests_django(session, django):
    session.install(f"Django>={django},<{float(django)+1}")
    session.install("pytest", "pytest-django")
    session.run("pytest")


@nox.session(python=False)
def lint(session):
    """Usa l'interprete di sistema senza creare un virtualenv."""
    session.install("ruff")
    session.run("ruff", "check", ".")
```

`@nox.parametrize` genera una sessione per ogni combinazione. `python=False` usa l'ambiente di sistema (utile per strumenti di sistema).

## Tox vs Nox

| Caratteristica | Tox | Nox |
|---------------|-----|-----|
| Configurazione | `tox.ini` dichiarativo | `noxfile.py` Python |
| Curva di apprendimento | Bassa | Media |
| Flessibilità | Limitata | Massima |
| Matrice ambienti | Sintassi graffe | `@nox.parametrize` |
| Generazione ambienti | Automatica | Automatica |
| CI integration | Eccellente | Eccellente |
| Caso d'uso tipico | Test multi-versione standard | Pipeline complesse, tooling |

## Integrazione CI (GitHub Actions)

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]

jobs:
  tox:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11"]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install Tox
        run: pip install tox
      - name: Run Tox
        run: tox
```

Sia Tox che Nox si integrano nativamente con CI. Il pattern tipico: `pip install tox && tox`.

## Errori comuni

- **Tox non trova l'interprete Python**: installa la versione mancante o usa `skip_missing_interpreters = true`.
- **Dipendenze non aggiornate**: Tox/Nox creano ambienti da zero. Usa `-r` o `recreate` per forzare ricreazione.
- **Nox `session.install` posizionale**: l'ordine conta per risolvere dipendenze conflittuali.
- **Confondere Tox e Nox in CI**: decidine uno e usa solo quello. Supportano entrambi le stesse features.
- **Dimenticare `.github/workflows/` syntax**: Tox/Nox sono runner locali, CI li esegue in un ambiente pulito.

## Best Practices & Conventions

- Usa **Tox** per progetti standard con test multi-versione e linting. È più semplice.
- Usa **Nox** se hai bisogno di logiche condizionali, parametri complessi, o generazione di artefatti.
- Mantieni la configurazione versionata con il codice (`tox.ini` o `noxfile.py` in root).
- Combina con **pytest** e **coverage** per metriche complete.
- Includi sempre un ambiente `lint` con ruff o flake8.
- In CI, esegui Tox/Nox in una matrice di OS e versioni Python.
