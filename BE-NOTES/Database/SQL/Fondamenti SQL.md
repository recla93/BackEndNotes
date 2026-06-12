# Fondamenti SQL

**CRUD** = Create, Read, Update, Delete — le 4 operazioni fondamentali sui dati.

## CREATE (INSERT)

```sql
-- Singola riga
INSERT INTO persona (nome, cognome, eta) VALUES ('Mario', 'Rossi', 30);

-- Multipla righe
INSERT INTO persona (nome, cognome, eta) VALUES
    ('Mario', 'Rossi', 30),
    ('Luigi', 'Verdi', 25),
    ('Anna', 'Bianchi', 28);

-- Inserire tutte le colonne (senza specificarle — sconsigliato, fragile)
INSERT INTO persona VALUES (1, 'Mario', 'Rossi', 30);
```

## READ (SELECT)

```sql
-- Tutte le colonne
SELECT * FROM persona;

-- Colonne specifiche
SELECT nome, cognome FROM persona;

-- Con alias
SELECT nome AS nome_utente, eta AS age FROM persona;
```

### WHERE — filtrare righe

```sql
-- Operatori di confronto
SELECT * FROM persona WHERE eta > 25;
SELECT * FROM persona WHERE eta >= 20 AND eta <= 30;
SELECT * FROM persona WHERE cognome = 'Rossi' OR cognome = 'Verdi';
SELECT * FROM persona WHERE eta != 30;

-- BETWEEN — intervallo inclusivo
SELECT * FROM persona WHERE eta BETWEEN 20 AND 30;

-- IN — insieme di valori
SELECT * FROM persona WHERE cognome IN ('Rossi', 'Verdi', 'Bianchi');

-- LIKE — pattern matching
SELECT * FROM persona WHERE cognome LIKE 'R%';    -- Inizia con R
SELECT * FROM persona WHERE cognome LIKE '%i';    -- Finisce con i
SELECT * FROM persona WHERE nome LIKE '_ario';    -- _ = esattamente un carattere

-- IS NULL / IS NOT NULL
SELECT * FROM persona WHERE email IS NULL;
SELECT * FROM persona WHERE email IS NOT NULL;
```

### ORDER BY — ordinamento

```sql
SELECT * FROM persona ORDER BY cognome;           -- ASC (default)
SELECT * FROM persona ORDER BY eta DESC;           -- Discendente
SELECT * FROM persona ORDER BY cognome ASC, nome ASC;
```

### LIMIT e OFFSET — paginazione

```sql
SELECT * FROM persona LIMIT 10;                    -- prime 10 righe
SELECT * FROM persona LIMIT 10 OFFSET 20;          -- righe 21-30
SELECT * FROM persona ORDER BY id LIMIT 10 OFFSET 0;
```

### DISTINCT — valori unici

```sql
SELECT DISTINCT cognome FROM persona;
SELECT DISTINCT cognome, nome FROM persona;  -- combinazione unica
```

### Funzioni di aggregazione

```sql
SELECT COUNT(*) FROM persona;                    -- numero totale righe
SELECT COUNT(email) FROM persona;                 -- conteggio non-null
SELECT AVG(eta) FROM persona;                     -- media
SELECT SUM(eta) FROM persona;                     -- somma
SELECT MAX(eta) FROM persona;                     -- massimo
SELECT MIN(eta) FROM persona;                     -- minimo
```

### GROUP BY e HAVING

```sql
-- Raggruppa per cognome e conta
SELECT cognome, COUNT(*) AS quanti
FROM persona
GROUP BY cognome;

-- Filtra sui gruppi (HAVING, non WHERE!)
SELECT cognome, COUNT(*) AS quanti
FROM persona
GROUP BY cognome
HAVING COUNT(*) > 1;
```

## UPDATE

```sql
-- Aggiornare una riga
UPDATE persona SET eta = 31 WHERE nome = 'Mario';

-- Aggiornare multiple colonne
UPDATE persona SET eta = 31, cognome = 'Rossi' WHERE nome = 'Mario';

-- ⚠️ SENZA WHERE aggiorna TUTTE le righe!
UPDATE persona SET eta = 25;  -- PERICOLO!
```

## DELETE

```sql
-- Eliminare riga specifica
DELETE FROM persona WHERE id = 1;

-- Eliminare multiple righe
DELETE FROM persona WHERE eta < 18;

-- ⚠️ SENZA WHERE elimina TUTTE le righe!
DELETE FROM persona;  -- PERICOLO!

-- TRUNCATE: svuota la tabella + resetta auto-increment (più veloce di DELETE)
TRUNCATE TABLE persona;
```

## Differenza DELETE vs TRUNCATE vs DROP

| Comando | Cosa fa | WHERE | Rollback | Velocità |
|---|---|---|---|---|
| `DELETE` | Elimina righe | ✓ (opzionale) | ✓ (se in transazione) | Lenta (row by row) |
| `TRUNCATE` | Svuota tabella | ✗ | Dipende dal DB | Molto veloce |
| `DROP` | Elimina tabella | ✗ | Dipende dal DB | Immediata |

## Ordine di esecuzione di una SELECT

```sql
SELECT cognome, COUNT(*)          -- 5. Seleziona colonne
FROM persona                       -- 1. Tabella
WHERE eta > 18                     -- 2. Filtra righe
GROUP BY cognome                   -- 3. Raggruppa
HAVING COUNT(*) > 1                -- 4. Filtra gruppi
ORDER BY cognome                   -- 6. Ordina
LIMIT 10;                          -- 7. Limita
```

## Operatori WHERE: tabella riassuntiva

| Operatore | Esempio | Significato |
|---|---|---|
| `=` | `eta = 30` | Uguale |
| `!=` o `<>` | `eta != 30` | Diverso |
| `>` | `eta > 30` | Maggiore |
| `<` | `eta < 30` | Minore |
| `>=` | `eta >= 30` | Maggiore o uguale |
| `<=` | `eta <= 30` | Minore o uguale |
| `BETWEEN` | `eta BETWEEN 20 AND 30` | Intervallo (inclusivo) |
| `IN` | `cognome IN ('Rossi', 'Verdi')` | Insieme di valori |
| `LIKE` | `nome LIKE 'M%'` | Pattern matching (`%`=qualsiasi, `_`=esattamente uno) |
| `IS NULL` | `email IS NULL` | È nullo |
| `AND` | `eta > 20 AND eta < 30` | Entrambe le condizioni |
| `OR` | `eta < 20 OR eta > 60` | Almeno una condizione |
| `NOT` | `cognome NOT IN ('Rossi')` | Negazione |