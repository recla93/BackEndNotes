**CRUD** = Create, Read, Update, Delete

### CREATE (INSERT)

```sql
-- Inserire una singola riga
INSERT INTO persona (nome, cognome, eta)
VALUES ('Mario', 'Rossi', 30);

-- Inserire più righe
INSERT INTO persona (nome, cognome, eta) VALUES
('Mario', 'Rossi', 30),
('Luigi', 'Verdi', 25),
('Anna', 'Bianchi', 28);
```

### READ (SELECT)

```sql
-- Tutte le colonne di tutte le righe
SELECT * FROM persona;

-- Colonne specifiche
SELECT nome, cognome FROM persona;

-- Con condizione
SELECT * FROM persona WHERE eta > 25;

-- Più condizioni
SELECT * FROM persona 
WHERE eta >= 20 AND eta <= 30;

-- OR
SELECT * FROM persona 
WHERE cognome = 'Rossi' OR cognome = 'Verdi';

-- DISTINCT (valori unici)
SELECT DISTINCT cognome FROM persona;
```

### UPDATE

```sql
-- Aggiornare colonne
UPDATE persona SET eta = 31 WHERE nome = 'Mario';

-- Aggiornare multiple colonne
UPDATE persona 
SET eta = 31, cognome = 'Rossi' 
WHERE nome = 'Mario';

-- Attenzione: senza WHERE aggiorna TUTTE le righe
UPDATE persona SET eta = 25;  -- ⚠️ Pericolo!
```

### DELETE

```sql
-- Eliminare riga specifica
DELETE FROM persona WHERE id = 1;

-- Eliminare multiple righe
DELETE FROM persona WHERE eta < 18;

-- Attenzione: senza WHERE elimina TUTTE le righe
DELETE FROM persona;  -- ⚠️ Pericolo!
```