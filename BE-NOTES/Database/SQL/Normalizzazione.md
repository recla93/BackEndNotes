# Normalizzazione

La normalizzazione è il processo di **organizzazione dei dati** per ridurre ridondanza e dipendenze inconsistenti.

## Prima Forma Normale (1NF)

Ogni colonna contiene **un solo valore atomico** — niente liste o valori multipli in una cella.

| id | nome | materie | ❌ 1NF violata |
|---|---|---|---|
| 1 | Mario | Matematica, Italiano | |

```sql
-- ❌ VIOLAZIONE: colonna con valori multipli
CREATE TABLE studente (
    id INT,
    nome VARCHAR(100),
    materie VARCHAR(500)  -- "Matematica, Italiano, Inglese"
);

-- ✅ 1NF: ogni valore in una riga separata
CREATE TABLE studente (id INT, nome VARCHAR(100));
CREATE TABLE materia_studente (
    id_studente INT,
    id_materia INT,
    FOREIGN KEY (id_studente) REFERENCES studente(id)
);
```

## Seconda Forma Normale (2NF)

**1NF +** ogni colonna non-chiave dipende dall'**intera** chiave primaria (non solo da una parte).

```sql
-- ❌ VIOLAZIONE: ordine_id + prodotto_id = chiave composta
-- nome_prodotto dipende solo da prodotto_id, non dall'intera chiave
CREATE TABLE ordine_dettaglio (
    ordine_id INT,
    prodotto_id INT,
    quantita INT,
    nome_prodotto VARCHAR(100),  -- dipende solo da prodotto_id!
    PRIMARY KEY (ordine_id, prodotto_id)
);

-- ✅ 2NF: nome_prodotto spostato nella tabella prodotto
CREATE TABLE ordine_dettaglio (
    ordine_id INT,
    prodotto_id INT,
    quantita INT,
    PRIMARY KEY (ordine_id, prodotto_id)
);
CREATE TABLE prodotto (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);
```

## Terza Forma Normale (3NF)

**2NF +** non ci sono **dipendenze transitive** — ogni colonna non-chiave dipende **solo** dalla chiave primaria.

```sql
-- ❌ VIOLAZIONE: cliente_nome dipende da cliente_id, non da ordine.id
CREATE TABLE ordine (
    id INT PRIMARY KEY,
    cliente_id INT,
    cliente_nome VARCHAR(100),  -- dipendenza transitiva!
    cliente_email VARCHAR(100)
);

-- ✅ 3NF: dati del cliente separati
CREATE TABLE ordine (id INT PRIMARY KEY, cliente_id INT);
CREATE TABLE cliente (id INT PRIMARY KEY, nome VARCHAR(100), email VARCHAR(100));
```

## Single Source Of Truth (SSOT)

Un dato deve stare **in un solo posto**:

```sql
-- ❌ Stessa info in due tabelle diverse
UPDATE studente SET nome = 'Mario' WHERE id = 1;
UPDATE persona SET nome = 'Mario' WHERE id = 1;
-- Se dimentichi una...
```

## Quando NON normalizzare

La normalizzazione spinta (oltre 3NF) può essere **controproducente**:

| Situazione | Meglio denormalizzare |
|---|---|
| **Report / Data Warehouse** | JOIN troppi = lento. Preferisci dati ridondanti ma veloci |
| **Performance critiche** | Poche scritture, molte letture. Denormalizza per evitare JOIN |
| **Cache** | Copie di dati per velocizzare letture |
| **Log / Audit** | Dati storici che non cambiano mai |

**Regola pratica:** normalizza fino alla 3NF per il DB transazionale (OLTP). Denormalizza per query e report (OLAP).

## Riepilogo forme normali

| Forma | Requisito | Esempio violazione |
|---|---|---|
| **1NF** | Valori atomici | Colonna "materie" con "Matematica, Italiano" |
| **2NF** | Dipendenza totale dalla chiave | Chiave composta (ordine+prodotto), colonna dipende solo da prodotto |
| **3NF** | Nessuna dipendenza transitiva | Ordine con nome cliente (dipende da cliente_id, non da ordine_id) |

Oltre la 3NF esistono BCNF, 4NF, 5NF — ma nella pratica si arriva raramente oltre la 3NF.