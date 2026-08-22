---
topic: "Pandas"
tags: [python, data, pandas, dataframe]
nav_prev: "[[Alembic.md]]"
---
Riferimento ufficiale: [pandas.pydata.org/docs](https://pandas.pydata.org/docs/)

Pandas è la libreria standard per data analysis in Python. Strutture dati principali: `Series` (vettore 1D) e `DataFrame` (tabella 2D).

Pandas è pensato per **data in memoria** (fino a ~RAM, non oltre). Per dataset che superano la RAM, usa Dask, Polars o database query.

### Performance note
- **Evita loop su righe** (`iterrows`, `apply` con axis=1) — vettorizza
- **Preferisci `merge` a `join`** (più esplicito)
- **`category` dtype** per colonne con pochi valori unici
- **`inplace=True`** è deprecato, usa assegnazione: `df = df.rename(...)`

Vedi anche: [[BE-NOTES/Python/Data/Database Connection|Database]] per leggere DB in DataFrame, [[BE-NOTES/Python/Core Concepts/File e IO|File I/O]] per formati CSV/JSON.

`pd.Series` è un array 1D con indice personalizzabile — l'indice non è 0..n ma può essere stringhe, date, o qualsiasi hashable. `pd.DataFrame` è una tabella 2D dove ogni colonna è una Series. `read_csv`/`read_json` parsano file direttamente in DataFrame. Da dict list, ogni chiave diventa una colonna, ogni dict una riga.

```python
import pandas as pd

# Series — vettore con indice
s = pd.Series([10, 20, 30], index=["a", "b", "c"])

# DataFrame — tabella 2D
df = pd.DataFrame({
    "nome": ["Mario", "Luigi", "Anna"],
    "eta": [25, 30, 22],
    "città": ["Roma", "Milano", "Napoli"],
})

# Da CSV
df = pd.read_csv("dati.csv")

# Da JSON
df = pd.read_json("dati.json")

# Da dict list
records = [{"nome": "Mario", "eta": 25}, {"nome": "Anna", "eta": 30}]
df = pd.DataFrame(records)
```

## Ispezione

`.head(n)` mostra le prime n righe (default 5) — primo passo dopo un caricamento per vedere se i dati sono corretti. `.info()` mostra dtypes e conteggio non-null per colonna — rileva immediatamente dati mancanti o tipi sbagliati. `.describe()` produce statistiche descrittive (count, mean, std, min, quartili, max) per colonne numeriche. `.value_counts()` conta le occorrenze di ogni valore unico — ideale per colonne categoriche.

```python
df.head(10)          # prime righe
df.info()            # tipos, non-null
df.describe()        # statistiche
df.shape             # (righe, colonne)
df.columns           # nomi colonne
df.dtypes            # tipos per colonna
df["colonna"].value_counts()  # frequenze
```

## Selezione e filtro

`df["colonna"]` restituisce una Series (1D); `df[["col1", "col2"]]` restituisce un DataFrame (2D). `.iloc[]` seleziona per posizione intera (0 = prima riga); `.loc[]` per etichetta dell'indice. `df[df["eta"] > 25]` è il filtro booleano: l'espressione interna produce una Series di bool, usata come maschera sulle righe. `.query()` accetta una stringa di espressione — più leggibile per filtri multipli.

```python
# Colonna
df["nome"]                  # Series
df[["nome", "eta"]]         # DataFrame (multi-colonna)

# Riga per indice
df.iloc[0]                  # prima riga (position)
df.loc[0]                   # riga per indice label
df.iloc[1:5]                # slicing righe (position)

# Filtro booleano
df[df["eta"] > 25]
df[(df["eta"] >= 18) & (df["città"] == "Roma")]
df.query("eta > 25 & città == 'Roma'")
```

## Operazioni comuni

`df["colonna"] = ...` aggiunge o sovrascrive una colonna — le operazioni sono vettorizzate (nessun loop in Python). `groupby("città")["eta"].mean()` raggruppa per città e calcola la media dell'età per gruppo: equivale a SQL `SELECT città, AVG(eta) FROM df GROUP BY città`. `.agg()` permette multiple aggregazioni su più colonne. `.str.upper()` è un accessor che applica metodi stringa vettorizzati. `.apply()` con lambda è utile ma **più lento** della vettorizzazione — usalo solo quando non c'è alternativa nativa.

```python
# Nuova colonna
df["età_doppia"] = df["eta"] * 2

# GroupBy / aggregazione
df.groupby("città")["eta"].mean()
df.groupby("città").agg({"eta": ["mean", "max", "count"]})

# Ordinamento
df.sort_values("eta", ascending=False)

# Drop NA
df.dropna()
df.dropna(subset=["email"])

# Fill NA
df.fillna(0)
df["eta"].fillna(df["eta"].mean())

# Apply funzione
df["nome_maiuscolo"] = df["nome"].str.upper()
df["eta_quadrupla"] = df["eta"].apply(lambda x: x * 4)
```

## Merge e join (come SQL)

`pd.merge()` equivale a JOIN SQL: `on="id"` specifica la colonna chiave, `how` determina il tipo di join. A differenza di SQL, Pandas non ha indici — fa match su colonne con scansione/hash. `pd.concat()` unisce righe (UNION ALL SQL): `ignore_index=True` resetta l'indice invece di mantenere quelli originali.

```python
# SQL: INNER JOIN
pd.merge(df1, df2, on="id", how="inner")

# LEFT JOIN
pd.merge(df1, df2, on="id", how="left")

# Concat — unisce righe
pd.concat([df1, df2], ignore_index=True)
```

## Esportazione

`index=False` evita di scrivere l'indice come colonna nel CSV (altrimenti avresti una colonna Unnamed). `orient="records"` produce una lista di dict: `[{"col": val}, ...]` — il formato più comune per JSON. `to_excel` richiede `openpyxl` o `xlsxwriter` installati.

```python
df.to_csv("output.csv", index=False)
df.to_json("output.json", orient="records", indent=2)
df.to_excel("output.xlsx", sheet_name="Foglio1")
```

## Errori comuni

| Errore | Causa | Fix |
|---|---|---|
| `SettingWithCopyWarning` | Modifica di una fetta (copia) invece del DataFrame originale | Usa `.loc[...]` o `.copy()` esplicito |
| `KeyError` su colonna | Nome colonna non trovato (case-sensitive) | Controlla `df.columns` per i nomi esatti |
| MemoryError con dataset grande | DataFrame supera la RAM disponibile | Usa chunking (`read_csv(chunksize=)`) o Dask/Polars |
| `.apply()` lentissimo | Usato su migliaia di righe con axis=1 | Vettorizza con operazioni native Pandas |
| Merge restituisce più righe del previsto | Chiave non univoca in uno dei due DataFrame | Verifica duplicati con `duplicated()`
