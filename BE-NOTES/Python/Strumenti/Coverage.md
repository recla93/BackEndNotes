---
topic: "Coverage (Copertura Test) — Python"
tags: [python, testing, coverage, pytest-cov, quality]
---
Riferimento ufficiale: [coverage.readthedocs.io](https://coverage.readthedocs.io/)

Coverage.py misura quali linee di codice vengono eseguite durante i test. La copertura è espressa in percentuale: linee coperte / linee totali. Non indica che il codice sia corretto, ma che è stato **eseguito** almeno una volta.

La copertura va usata come **indicatore**, non come obiettivo. 100% di copertura non significa 0 bug. Lo scopo è trovare codice non testato, non inseguire un numero.

Vedi anche:
[[BE-NOTES/Python/Strumenti/pytest|pytest]],
[[BE-NOTES/Python/Strumenti/Tox e Nox|Tox e Nox]],
[[BE-NOTES/Python/Strumenti/Formattazione e Lint|Formattazione e Lint]].

## Uso base

```bash
pip install coverage

# Esecuzione con coverage
coverage run -m pytest

# Report testuale
coverage report

# Report HTML (apri htmlcov/index.html nel browser)
coverage html
```

`coverage run -m pytest` esegue i test sotto copertura. Il report mostra per ogni file: linee coperte, mancanti, percentuale. Il report HTML evidenzia in rosso le linee non eseguite.

## Configurazione

```ini
# .coveragerc
[run]
source = mio_package
omit =
    */tests/*
    */migrations/*
    */setup.py

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise NotImplementedError
    if __name__ == .__main__.:
    if TYPE_CHECKING:

[html]
directory = htmlcov
```

`source` limita la misurazione al tuo codice (esclude librerie). `omit` esclude cartelle dai report. `exclude_lines` ignora linee marcate o pattern noti.

## Integrazione con pytest

```bash
# Usa pytest-cov (pip install pytest-cov)
pytest --cov=mio_package tests/
pytest --cov=mio_package --cov-report=html --cov-report=term
```

`pytest-cov` integra coverage direttamente in pytest. `--cov-report=term` (default), `--cov-report=html`, `--cov-report=xml` (per CI).

## Condizionali e path coverage

```python
def calcola_sconto(prezzo: float, fedelta: bool) -> float:
    if fedelta:
        return prezzo * 0.9       # branch True
    return prezzo                  # branch False
```

Coverage.py traccia sia **line coverage** (quali righe) che **branch coverage** (quali rami condizionali). Abilita con `--branch-mixin` o `[run] branch = True`. Nel report, le linee con branch parziale sono marcate con `->`.

## Escludere linee dal conteggio

```python
class MioCodice:
    def complessa(self):
        pass  # pragma: no cover

# Escludere blocchi interi
if __name__ == "__main__":  # pragma: no cover
    main()
```

`# pragma: no cover` esclude la riga successiva o il blocco indentato. Usalo con parsimonia per: codici di debug, blocchi `if TYPE_CHECKING`, `__repr__` banali, rami `except` difficili da simulare.

## Failure threshold in CI

```bash
# Fallisce se copertura < 80%
coverage report --fail-under=80
```

```yaml
# GitHub Actions
- name: Check coverage
  run: |
    coverage run -m pytest
    coverage report --fail-under=80
```

Imposta una soglia minima in CI. Sotto la soglia, il job fallisce. Soglia tipica: 80% per progetti nuovi, 60-70% per legacy con debito tecnico.

## File .coveragerc esempi avanzati

```ini
[run]
source = mio_package
branch = True
parallel = True
concurrency = multiprocessing

[report]
sort = Cover
skip_covered = True
exclude_lines =
    pragma: no cover
    def __repr__
    if self.debug:
    if TYPE_CHECKING:

[paths]
source =
    src/
    .tox/*/site-packages/
```

`parallel = True` serve per test paralleli (pytest-xdist). `[paths]` mappa percorsi tra ambienti diversi (tox). `skip_covered = True` nasconde file al 100%.

## Errori comuni

- **100% è una trappola**: codice coperto ma non testato nei comportamenti giusti. Un test che chiama una funzione senza assert dà copertura ma zero garanzia.
- **Dimenticare `source`**: senza, coverage misura anche librerie di sistema. Report gonfiati.
- **Ignorare i branch**: 100% line coverage può nascondere branch non testati. Abilita branch coverage.
- **Troppi pragma: no cover**: nascondono il problema invece di risolverlo. Usali solo per eccezioni legittime.
- **Confondere copertura con qualità**: coverage è uno strumento, non un obiettivo. Scrivi test per comportamento, non per percentuale.

## Best Practices & Conventions

- Imposta una **soglia minima** in CI (80% è un buon default).
- Abilita **branch coverage** per metriche più accurate.
- Usa **`--cov-report=html`** per review visuali durante il development.
- Combina con **Tox/Nox** per eseguire coverage su più versioni Python.
- Non ossessionarti sulla percentuale: concentrati su rami critici e casi limite non coperti.
- Revisiona periodicamente il report HTML per identificare codice morto o non testato.
