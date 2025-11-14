### Primary Key (Chiave Primaria)

La **chiave primaria** identifica univocamente ogni riga:

```sql
CREATE TABLE persona (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    cognome VARCHAR(100)
);
```

**Proprietà**:
- ✓ Unica (no duplicati)
- ✓ Non nulla (NOT NULL)
- ✓ Una sola per tabella
- ✓ Identifica univocamente la riga

### Foreign Key (Chiave Esterna)

La **chiave esterna** crea una relazione tra tabelle, mantenendo l'integrità referenziale:

```sql
CREATE TABLE provincia (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    id_regione INT,
    FOREIGN KEY (id_regione) REFERENCES regione(id)
);
```

### Relazione 1-N (Uno a Molti)

Una **regione** ha **molte** province:

```
regione (1) ----------- (N) provincia
```

La chiave esterna sta nella tabella "molti" (provincia):

```sql
-- Tabella Regione
CREATE TABLE regione (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);

-- Tabella Provincia (figlio)
CREATE TABLE provincia (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    id_regione INT,
    FOREIGN KEY (id_regione) REFERENCES regione(id)
);
```

**Query di JOIN**:
```sql
SELECT r.nome as regione, p.nome as provincia
FROM regione r
JOIN provincia p ON r.id = p.id_regione;
```

### Relazione N-N (Molti a Molti)

**Docenti** insegnano **molte** materie, e **materie** sono insegnate da **molti** docenti:

```
docente (N) --------- (N) materia
```

Serve una **tabella associativa** nel mezzo:

```sql
CREATE TABLE docente (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);

CREATE TABLE materia (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);

-- Tabella associativa
CREATE TABLE docente_materia (
    id_docente INT,
    id_materia INT,
    PRIMARY KEY (id_docente, id_materia),
    FOREIGN KEY (id_docente) REFERENCES docente(id) ON DELETE CASCADE,
    FOREIGN KEY (id_materia) REFERENCES materia(id) ON DELETE CASCADE
);
```

**Query di JOIN**:
```sql
-- Tutti i docenti di una materia
SELECT d.nome
FROM docente d
JOIN docente_materia dm ON d.id = dm.id_docente
JOIN materia m ON m.id = dm.id_materia
WHERE m.nome = 'Matematica';
```

### Primary Key (Chiave Primaria)

La **chiave primaria** identifica univocamente ogni riga:

```sql
CREATE TABLE persona (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    cognome VARCHAR(100)
);
```

**Proprietà**:
- ✓ Unica (no duplicati)
- ✓ Non nulla (NOT NULL)
- ✓ Una sola per tabella
- ✓ Identifica univocamente la riga

### Foreign Key (Chiave Esterna)

La **chiave esterna** crea una relazione tra tabelle, mantenendo l'integrità referenziale:

```sql
CREATE TABLE provincia (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    id_regione INT,
    FOREIGN KEY (id_regione) REFERENCES regione(id)
);
```

### Relazione 1-N (Uno a Molti)

Una **regione** ha **molte** province:

```
regione (1) ----------- (N) provincia
```

La chiave esterna sta nella tabella "molti" (provincia):

```sql
-- Tabella Regione
CREATE TABLE regione (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);

-- Tabella Provincia (figlio)
CREATE TABLE provincia (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    id_regione INT,
    FOREIGN KEY (id_regione) REFERENCES regione(id)
);
```

**Query di JOIN**:
```sql
SELECT r.nome as regione, p.nome as provincia
FROM regione r
JOIN provincia p ON r.id = p.id_regione;
```

### Relazione N-N (Molti a Molti)

**Docenti** insegnano **molte** materie, e **materie** sono insegnate da **molti** docenti:

```
docente (N) --------- (N) materia
```

Serve una **tabella associativa** nel mezzo:

```sql
CREATE TABLE docente (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);

CREATE TABLE materia (
    id INT PRIMARY KEY,
    nome VARCHAR(100)
);

-- Tabella associativa
CREATE TABLE docente_materia (
    id_docente INT,
    id_materia INT,
    PRIMARY KEY (id_docente, id_materia),
    FOREIGN KEY (id_docente) REFERENCES docente(id) ON DELETE CASCADE,
    FOREIGN KEY (id_materia) REFERENCES materia(id) ON DELETE CASCADE
);
```

**Query di JOIN**:
```sql
-- Tutti i docenti di una materia
SELECT d.nome
FROM docente d
JOIN docente_materia dm ON d.id = dm.id_docente
JOIN materia m ON m.id = dm.id_materia
WHERE m.nome = 'Matematica';
```

