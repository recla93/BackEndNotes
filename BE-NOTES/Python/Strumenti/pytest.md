---
topic: "pytest"
tags: [python, testing, pytest, tdd]
nav_next: "[[Formattazione e Lint.md]]"
---
Riferimento ufficiale: [docs.pytest.org](https://docs.pytest.org/)

pytest è il framework di test standard per Python. A differenza di `unittest` (Java JUnit-style), pytest usa **`assert` nativo** (nessuna `assertEqual`), **fixtures** per setup/teardown, **parametrizzazione** built-in.

### Perché pytest > unittest

| pytest | unittest |
|---|---|
| `assert x == y` | `self.assertEqual(x, y)` |
| Fixtures funzionali | setUp/tearDown |
| Parametrizzazione integrata | SubTest/subclass |
| Plugin ecosystem | Minimo |
| Auto-discovery | Manuale |

Vedi anche: [[BE-NOTES/Python/FastAPI/Testing e Deploy|FastAPI TestClient]], [[BE-NOTES/Python/Strumenti/Formattazione e Lint|Lint]] per CI integration.

pytest esegue **auto-discovery** dei test: cerca file `test_*.py` o `*_test.py`, poi funzioni `test_*` e classi `Test*`. `-v` (verbose) mostra ogni test singolarmente. `-k "test_utente"` filtra per nome — utile per eseguire solo test rilevanti. `--cov=app` misura la coverage del modulo `app` (richiede `pytest-cov`).

```bash
pip install pytest pytest-cov pytest-mock
# Eseguire:
pytest
pytest -v                    # verbose
pytest -k "test_utente"      # filtra per nome
pytest --cov=app test/       # coverage
```

## Test semplici

pytest usa `assert` Python nativo — nessun bisogno di `self.assertEqual()`. Se il test fallisce, pytest mostra il valore dell'espressione: `assert 2 + 2 == 5` mostra `E  assert (2 + 2) == 5`. `assert not []` e `assert [1]` usano la truthiness Python: liste vuote sono falsy, liste non vuote truthy.

```python
# test_math.py
def test_somma():
    assert 2 + 2 == 4

def test_sottrazione():
    assert 5 - 3 == 2

def test_valore_vuoto():
    assert not []
    assert [1]  # truthy
```

## Test con eccezioni

`pytest.raises(ValueError, match="negativo")` verifica che il codice lanci `ValueError` e che il messaggio contenga "negativo". Se non viene lanciata eccezione, il test fallisce. `match` accetta regex — utile per distinguere tra eccezioni dello stesso tipo con messaggi diversi.

```python
import pytest

def test_errore():
    with pytest.raises(ValueError, match="negativo"):
        funzione_che_lancia(-1)
```

## Fixtures

Le fixtures sono funzioni che preparano dati/oggetti per i test. `@pytest.fixture` senza parametri ha `scope="function"` (eseguita per ogni test). Con `yield` invece di `return`, il codice dopo `yield` è il teardown (eseguito anche se il test fallisce). `scope="session"` esegue la fixture una volta per l'intera run di pytest — utile per connessioni DB costose. Le fixtures sono **iniettate** nei test per nome del parametro.

```python
import pytest

@pytest.fixture
def dati_test():
    """Setup: crea dati per i test."""
    return {"nome": "Mario", "eta": 25}

@pytest.fixture
def db_session():
    """Setup/teardown con yield."""
    db = connessione_test()
    yield db
    db.chiudi()  # cleanup

def test_utente(dati_test):
    assert dati_test["nome"] == "Mario"

# Fixture con scope
@pytest.fixture(scope="session")   # una volta per sessione
@pytest.fixture(scope="module")    # una volta per modulo
@pytest.fixture(scope="class")     # una volta per classe
@pytest.fixture(scope="function")  # default: ogni test
```

## Parametrizzazione

`@pytest.mark.parametrize("input,expected", [...])` genera un test separato per ogni tupla — se il terzo caso fallisce, gli altri 3 vengono comunque eseguiti (non si ferma al primo fallimento). Multipli `@parametrize` si combinano in prodotto cartesiano: 2 valori di `a` × 2 di `b` = 4 test. Ogni combinazione ha un ID unico visibile con `-v`.

```python
@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (2, 4),
    (3, 6),
    (0, 0),
])
def test_doppio(input, expected):
    assert input * 2 == expected

# Combinazione di parametri
@pytest.mark.parametrize("a", [1, 2])
@pytest.mark.parametrize("b", [10, 20])
def test_somma(a, b):
    # Esegue 4 test: (1,10), (1,20), (2,10), (2,20)
    assert a + b == a + b
```

## Monkeypatch e Mock

`monkeypatch` è una fixture built-in di pytest per sostituire temporaneamente oggetti/funzioni. Il cambio è attivo solo dentro il test. `mocker` è fornito da `pytest-mock` (wrapper su `unittest.mock`): `mocker.patch("modulo.esterno.chiamata")` sostituisce l'oggetto nel namespace del modulo. `.return_value` imposta cosa restituisce la mock. `assert_called_once_with` verifica che sia stata chiamata con gli argomenti attesi.

```python
# monkeypatch — modifica oggetti temporaneamente
def test_api_call(monkeypatch):
    def mock_fetch(*args, **kwargs):
        return {"status": "ok"}

    monkeypatch.setattr("modulo.fetch_data", mock_fetch)
    risultato = modulo.processa()
    assert risultato["status"] == "ok"

# pytest-mock — interfaccia mock
def test_con_mock(mocker):
    mock = mocker.patch("modulo.esterno.chiamata")
    mock.return_value = {"id": 1}
    risultato = modulo.funzione()
    mock.assert_called_once_with("arg")
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `fixture not found` | Nome fixture errato o import mancante in `conftest.py` | Verifica nome e che `conftest.py` sia nella directory giusta |
| `ModuleNotFoundError` per codice sorgente | PYTHONPATH non include il modulo | Installa il pacchetto con `pip install -e .` |
| Test lento per fixture con scope="function" | Cone$$essione DB riaperta per ogni test | Usa `scope="session"` per risorse costose |
| `PytestAssertRewriteWarning` | `assert` in file non test (pytest non può riscriverlo) | Ignora o sposta la logica nei test |
| Mock non funziona | `mocker.patch` sul nome sbagliato (deve matchare il punto di import, non la definizione) | Patcha dove l'oggetto è USATO, non dove è DEFINITO |

## Struttura progetto

`conftest.py` è un file speciale: pytest lo carica automaticamente e rende le fixtures in esso definite disponibili a tutti i test nelle sottodirectory. Non serve importarlo esplicitamente. `__init__.py` in `test_api/` è opzionale (pytest la supporta comunque, ma aiuta l'ispezione IDE).

```
tests/
├── conftest.py         # fixtures condivise (auto-discoverate)
├── test_utenti.py
└── test_api/
    ├── __init__.py
    └── test_endpoint.py
```
