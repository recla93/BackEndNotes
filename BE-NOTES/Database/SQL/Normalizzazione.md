### Prima Forma Normale (1NF)

- Ogni colonna contiene **un solo valore** (no liste)
- Tutti i valori sono **atomici**

```sql
-- ❌ NON normalizzato
CREATE TABLE studente (
    id INT,
    nome VARCHAR(100),
    materie VARCHAR(500)  -- "Matematica, Italiano, Inglese"
);

-- ✅ Normalizzato
CREATE TABLE studente (
    id INT,
    nome VARCHAR(100)
);

CREATE TABLE materia_studente (
    id_studente INT,
    id_materia INT,
    FOREIGN KEY (id_studente) REFERENCES studente(id)
);
```

### Seconda Forma Normale (2NF)

- 1NF + ogni colonna dipende **interamente dalla chiave primaria**
- Non attributi dipendono solo da **parte della chiave**

### Terza Forma Normale (3NF)

- 2NF + non ci sono **dipendenze transitive**
- Le colonne dipendono **solo dalla chiave primaria**

```sql
-- ❌ Non normalizzato (dipendenza transitiva)
CREATE TABLE ordine (
    id INT,
    cliente_id INT,
    cliente_nome VARCHAR(100),  -- Dipende da cliente_id, non da ordine.id
    cliente_eta INT
);

-- ✅ Normalizzato
CREATE TABLE ordine (
    id INT,
    cliente_id INT,
    FOREIGN KEY (cliente_id) REFERENCES cliente(id)
);

CREATE TABLE cliente (
    id INT,
    nome VARCHAR(100),
    eta INT
);
```

### Single Source Of Truth (SSOT)

**Un dato deve stare in un solo posto** per evitare incongruenze:

```sql
-- ❌ MALE - Stessa info in due posti
UPDATE studente SET nome = 'Mario' WHERE id = 1;
UPDATE persona SET nome = 'Mario' WHERE id = 1;  -- Se dimentichi questo...

-- ✅ BENE - Info in un solo posto
-- Se studente eredita da persona, il nome è centralizzato
```