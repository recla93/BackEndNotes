---
topic: "Pydantic e Validazione — FastAPI"
tags: [python, fastapi, pydantic, validation]
nav_prev: "[[Setup e Primi Passi.md]]"
nav_next: "[[Dependency Injection.md]]"
---
Riferimento ufficiale: [docs.pydantic.dev](https://docs.pydantic.dev)

Pydantic è il cuore della validazione in FastAPI. Ogni request body, query parameter, path parameter viene validato automaticamente contro uno schema Pydantic.

A differenza di marshmallow o DRF serializers: tipi Python nativi, type hints, validazione con `@field_validator`, serializzazione automatica JSON.

Vedi anche: [[BE-NOTES/Python/FastAPI/Setup e Primi Passi|Setup]], [[BE-NOTES/Python/FastAPI/Auth e Sicurezza|Auth]] per schemi di login, [[BE-NOTES/Python/FastAPI/Database e SQLAlchemy|DB]] per schemi ORM.

`BaseModel` è la classe base di Pydantic. Quando crei una sottoclasse, Pydantic ispeziona i type hint e costruisce automaticamente un validatore per ogni campo. A runtime, `Utente(nome="...", eta=30)` valida ogni argomento contro il tipo dichiarato: `nome` deve essere `str`, `eta` deve essere `int` e inoltre ≥0 e ≤150 (grazie a `Field(ge=0, le=150)`), `email` deve matchare la regex. Se un campo ha un default (`città = "Roma"`), è opzionale. Se un campo è `None` (es. `creato`), accetta `null` in JSON. Se la validazione fallisce, Pydantic lancia `ValidationError` con tutti i dettagli — FastAPI lo trasforma in una risposta 422 con il campo incriminato.

```python
from pydantic import BaseModel, Field
from datetime import datetime

class Utente(BaseModel):
    nome: str
    eta: int = Field(ge=0, le=150)  # validazione range
    email: str = Field(pattern=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    città: str = "Roma"  # default
    creato: datetime | None = None
```

## Validazione Avanzata

```python
from pydantic import BaseModel, field_validator, model_validator

class Prenotazione(BaseModel):
    data_inizio: datetime
    data_fine: datetime

    @field_validator("data_fine")
    @classmethod
    def fine_dopo_inizio(cls, v: datetime, info) -> datetime:
        if "data_inizio" in info.data and v <= info.data["data_inizio"]:
            raise ValueError("data_fine deve essere dopo data_inizio")
        return v

    @model_validator(mode="after")
    def durata_massima(self):
        durata = (self.data_fine - self.data_inizio).days
        if durata > 30:
            raise ValueError("Prenotazione max 30 giorni")
        return self
```

## Query Parameters con Validazione

FastAPI integra Pydantic anche per i query parameter. `Query(...)` con `...` come primo argomento significa "obbligatorio". `Query(None)` o un valore di default rendono il parametro opzionale. `ge=0`, `min_length=1` sono validazioni Pydantic applicate al valore stringa estratto dalla query string. Se l'input non passa la validazione (es. `q` vuota o `limit=1000`), FastAPI risponde 422 prima ancora che la funzione venga eseguita.

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/search")
def search(
    q: str = Query(..., min_length=1, max_length=100),
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100),
):
    """... = obbligatorio, nessun default"""
    return {"query": q, "skip": skip, "limit": limit}

@app.get("/filtro")
def filtro(
    categoria: str | None = None,
    prezzo_max: float | None = Query(None, gt=0),
):
    return {"categoria": categoria, "prezzo_max": prezzo_max}
```

## Path Parameters con Validazione

Stessa logica dei query parameter, ma applicata a valori estratti dal path URL. `Path(ge=1)` significa che `item_id` nell'URL deve essere ≥1, altrimenti 422. Nota che la validazione avviene PRIMA della conversione di tipo: FastAPI estrae la stringa dal path, la valida, poi la converte in `int`.

```python
from fastapi import Path

@app.get("/items/{item_id}")
def get_item(
    item_id: int = Path(ge=1, title="ID dell'item"),
):
    return {"item_id": item_id}
```

## Config e conversione

`from_attributes = True` (Pydantic v2: `model_config = ConfigDict(from_attributes=True)`) abilita la creazione di modelli Pydantic da oggetti arbitrari (es. istanze SQLAlchemy). Senza questa configurazione, Pydantic accetta solo dict. Con `from_attributes=True`, puoi passare un'istanza ORM e Pydantic legge i suoi attributi — è il meccanismo standard per restituire dati DB come JSON via FastAPI.

```python
class UtenteInDB(BaseModel):
    nome: str
    eta: int

    class Config:
        from_attributes = True  # ORM mode (SQLAlchemy)

# Response model — filtra i campi in output
@app.get("/utente", response_model=Utente)
@app.post("/utente", response_model=Utente)
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `field required` | Campo obbligatorio mancante | Aggiungi default se opzionale |
| `type_error.integer` | Tipo sbagliato nel JSON | Invia numero invece di stringa |
| `from_attributes` non attivo | Pydantic v1 vs v2 | Usa `model_config = ConfigDict(from_attributes=True)` in v2 |
| Regex email troppo stretta | Pattern non copre tutti i casi | Usa `EmailStr` di `pydantic` se serve |

## Best practice

- **`Field()` per vincoli semplici**: `ge`, `le`, `min_length`, `pattern` bastano per l'80% dei casi
- **`model_validator` per regole multi-campo**: se coinvolge due campi (es. date), usa `model_validator` non `field_validator`
- **Separa schemi input/output**: `UtenteCreate` (input, con password) vs `UtenteOut` (output, senza password)
- **Pydantic v2**: per SQLAlchemy usa `model_config = ConfigDict(from_attributes=True)`
