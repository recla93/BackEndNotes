### Single Table Approach

Tutte le entità nella **stessa tabella**:

```sql
CREATE TABLE entita (
    id INT PRIMARY KEY,
    tipo VARCHAR(50),           -- "studente" o "docente"
    nome VARCHAR(100),
    eta INT,
    -- Campi solo per studente (NULL per docenti)
    numero_matricola INT,
    -- Campi solo per docente (NULL per studenti)
    stipendio DECIMAL
);
```

**Vantaggi**:
- ✓ Query semplici (no JOIN)
- ✓ Letture veloci

**Svantaggi**:
- ✗ Spreco di spazio (colonne NULL)
- ✗ Difficile da mantenere
- ✗ Poca integrità referenziale

### Class Table Inheritance (Tabelle per Classe)

**Tabella padre** + **tabelle figlie**:

```sql
CREATE TABLE persona (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    eta INT
);

CREATE TABLE studente (
    id INT PRIMARY KEY,
    numero_matricola INT,
    FOREIGN KEY (id) REFERENCES persona(id) ON DELETE CASCADE
);

CREATE TABLE docente (
    id INT PRIMARY KEY,
    stipendio DECIMAL,
    FOREIGN KEY (id) REFERENCES persona(id) ON DELETE CASCADE
);
```

**Vantaggi**:
- ✓ Nessuna ridondanza
- ✓ Integrità referenziale forte
- ✓ Facile da estendere

**Svantaggi**:
- ✗ Query più complesse (molti JOIN)
- ✗ Prestazioni ridotte

**Query di JOIN**:
```sql
-- Ottenere studenti con dati di persona
SELECT p.id, p.nome, p.eta, s.numero_matricola
FROM persona p
JOIN studente s ON p.id = s.id;

-- Con LEFT JOIN per includere tutti gli utenti
SELECT p.id, p.nome, p.eta, 
       s.numero_matricola, 
       d.stipendio
FROM persona p
LEFT JOIN studente s ON p.id = s.id
LEFT JOIN docente d ON p.id = d.id;
```

### Joined Table Inheritance

Molto simile a **Class Table Inheritance**. Ogni classe della gerarchia è memorizzata in una tabella separata, collegata via foreign key.
