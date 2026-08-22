---
topic: "Serializzazione (JSON, CSV, YAML) — Python"
tags: [python, serializzazione, json, csv, yaml, pickle]
---
Riferimento ufficiale JSON: [docs.python.org/3/library/json.html](https://docs.python.org/3/library/json.html)
Riferimento ufficiale CSV: [docs.python.org/3/library/csv.html](https://docs.python.org/3/library/csv.html)

La serializzazione è il processo di convertire oggetti Python in un formato salvabile/trasmissibile e viceversa. I formati più comuni in Python sono JSON (interoperabile, leggibile), CSV (tabelle, fogli di calcolo), YAML (configurazioni), e Pickle (formato binario Python-only).

La scelta del formato dipende dal caso d'uso: JSON per API, CSV per data export/import, YAML per configurazioni, Pickle per cache/caching di oggetti Python. Pickle è Python-only e insicuro da fonti non fidate.

Vedi anche:
[[BE-NOTES/Python/Core Concepts/File e IO|File e IO]],
[[BE-NOTES/Python/Data/Pandas|Pandas]],
[[BE-NOTES/Python/Strumenti/Docker per Python|Docker per Python]].

## JSON — JavaScript Object Notation

```python
import json

# Python -> JSON (serializzazione)
dati = {
    "nome": "Alice",
    "eta": 30,
    "attivo": True,
    "tags": ["admin", "editor"],
    "punteggio": None,
}

json_string = json.dumps(dati, indent=2, ensure_ascii=False)
print(json_string)

# JSON -> Python (deserializzazione)
ricaricato = json.loads(json_string)
print(ricaricato["nome"])  # Alice
```

`ensure_ascii=False` preserva i caratteri Unicode (es. accenti). `indent=2` formatta l'output leggibile. `json.dump()` e `json.load()` lavorano su file invece che stringhe.

## JSON — tipi non serializzabili

```python
import json
from datetime import datetime

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, set):
            return list(obj)
        if isinstance(obj, Decimal):
            return float(obj)
        return super().default(obj)

dati = {
    "data": datetime.now(),
    "set_di_valori": {1, 2, 3},
}

json_string = json.dumps(dati, cls=CustomEncoder)
```

`json` serializza solo tipi base (dict, list, str, int, float, bool, None). Per `datetime`, `Decimal`, `set`, `numpy.array` devi fornire un encoder personalizzato.

## CSV — Comma Separated Values

```python
import csv

# Lettura
with open("dati.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for riga in reader:
        print(riga["nome"], riga["eta"])

# Scrittura
with open("output.csv", "w", newline="", encoding="utf-8") as f:
    fieldnames = ["nome", "eta", "citta"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerow({"nome": "Alice", "eta": 30, "citta": "Roma"})
```

`newline=""` è necessario per evitare righe vuote extra su Windows. `DictReader`/`DictWriter` usano la prima riga come intestazione. Per file con delimitatore diverso (es. TSV), specifica `delimiter="\t"` o `delimiter=";"`.

## YAML — Yet Another Markup Language

```python
import yaml

# Lettura
with open("config.yaml", encoding="utf-8") as f:
    config = yaml.safe_load(f)

# Scrittura
config = {
    "server": {
        "host": "localhost",
        "port": 8080,
        "debug": True,
    },
    "database": {
        "url": "postgresql://localhost:5432/db",
        "pool_size": 10,
    },
}

with open("config.yaml", "w", encoding="utf-8") as f:
    yaml.dump(config, f, default_flow_style=False, allow_unicode=True)
```

PyYAML non fa parte della stdlib: `pip install pyyaml`. **Sempre** `yaml.safe_load()`, mai `yaml.load()` (esegue codice arbitrario se il file YAML contiene oggetti pericolosi).

## Pickle — formato binario Python

```python
import pickle

# Salvare
dati = {"complessi": [1, 2, 3], "data": datetime.now()}
with open("dati.pkl", "wb") as f:
    pickle.dump(dati, f)

# Caricare
with open("dati.pkl", "rb") as f:
    ricaricato = pickle.load(f)
```

Pickle salva **qualsiasi** oggetto Python, incluse classi e funzioni. Ma è **insicuro**: `pickle.load()` su file non fidato esegue codice arbitrario. Usalo solo per cache locali e comunicazione tra processi Python fidati.

## Formati minori

```python
import configparser

# Configparser per file .ini
config = configparser.ConfigParser()
config["DEFAULT"] = {"debug": "False", "port": "8080"}
with open("app.ini", "w") as f:
    config.write(f)
```

`configparser` è nella stdlib e utile per file `.ini` stile Windows. Per formati moderni, YAML o TOML sono preferibili.

## Tabella comparativa

| Formato | Stdlib | Leggibile | Tipi supportati | Sicurezza | Uso principale |
|---------|--------|-----------|-----------------|-----------|----------------|
| JSON | `json` | Si | Base | Sicuro | API, config |
| CSV | `csv` | Si | Stringhe/num. | Sicuro | Data export |
| YAML | No (pyyaml) | Si | Estesi | `safe_load()` | Config. complesse |
| Pickle | `pickle` | No | Tutti | **NO** | Cache interna |
| TOML | `tomllib` (3.11+) | Si | Base | Sicuro | Config Python (pyproject.toml) |
| INI | `configparser` | Si | Stringhe | Sicuro | Config legacy |

## Errori comuni

- **`JSONDecodeError`**: file JSON malformato. Validalo con `json.tool` prima: `python -m json.tool file.json`.
- **Dimenticare `ensure_ascii=False`**: caratteri non ASCII scritti come escape `\uXXXX`.
- **Usare `load()` invece di `safe_load()` in YAML**: vulnerabilità di code injection.
- **Pickle da fonti non fidate**: esecuzione di codice arbitrario. Mai caricare pickle da rete o input utente.
- **CSV con virgole nei campi**: usa `csv.QUOTE_ALL` o `csv.QUOTE_NONNUMERIC`.
- **Confondere `json.dumps()` (to string) con `json.dump()` (to file)**: l'ultima lettera `s` fa la differenza.

## Best Practices & Conventions

- Per API REST: **sempre JSON** con `ensure_ascii=False`.
- Per file di configurazione: **YAML con `safe_load()`** o **TOML** (standard Python per `pyproject.toml`).
- Per export/import tabellari: **CSV** con `DictReader`/`DictWriter`.
- Mai usare **Pickle** per scambio tra applicazioni o su rete.
- Per serializzazione di oggetti complessi, valuta `dataclasses_json` o `pydantic`.
- Specifica sempre `encoding="utf-8"` quando leggi/scivi file di testo.
