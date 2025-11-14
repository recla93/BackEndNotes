### Il Concetto di JOIN

Un **JOIN** unisce righe da tabelle diverse basandosi su una condizione (di solito uguaglianza tra chiavi):

```sql
-- Sintassi base
SELECT colonne
FROM tabella1
JOIN tabella2 ON tabella1.chiave = tabella2.chiave;
```

### INNER JOIN (Default JOIN)

Restituisce **solo le righe che hanno corrispondenza in entrambe le tabelle**:

```sql
SELECT p.nome, p.cognome, pr.nome as provincia
FROM persona p
INNER JOIN provincia pr ON p.provincia_id = pr.id;

-- Equivalente (senza INNER)
SELECT p.nome, p.cognome, pr.nome
FROM persona p
JOIN provincia pr ON p.provincia_id = pr.id;
```

**Diagramma**:
```
Tabella 1    Tabella 2
┌─────┐      ┌─────┐
│  A  │      │  A  │ ✓ Restituito
├─────┤      ├─────┤
│  B  │      │  C  │
├─────┤      ├─────┤
│  D  │      │  D  │ ✓ Restituito
└─────┘      └─────┘
  B ✗         (non ha corrispondenza)
```

### LEFT JOIN (LEFT OUTER JOIN)

Restituisce **tutte le righe della tabella sinistra** + righe corrispondenti della tabella destra (NULL se non ci sono):

```sql
SELECT p.nome, p.cognome, pr.nome as provincia
FROM persona p
LEFT JOIN provincia pr ON p.provincia_id = pr.id;

-- Include persone anche se non hanno provincia assegnata
```

**Diagramma**:
```
Tabella 1    Tabella 2
┌─────┐      ┌─────┐
│  A  │      │  A  │ ✓ Restituito
├─────┤      ├─────┤
│  B  │      │  C  │
├─────┤      ├─────┤
│  D  │      │  D  │ ✓ Restituito
└─────┘      └─────┘
  B ✓         (NULL)
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Il contrario di LEFT JOIN. Restituisce **tutte le righe della tabella destra** + righe corrispondenti della tabella sinistra:

```sql
SELECT p.nome, p.cognome, pr.nome as provincia
FROM persona p
RIGHT JOIN provincia pr ON p.provincia_id = pr.id;

-- Include tutte le province, anche quelle senza persone
```

### FULL OUTER JOIN

Restituisce **tutte le righe da entrambe le tabelle** (NULL dove non ci sono corrispondenze):

```sql
SELECT p.nome, p.cognome, pr.nome as provincia
FROM persona p
FULL OUTER JOIN provincia pr ON p.provincia_id = pr.id;

-- Nota: MySQL non supporta nativamente FULL JOIN
-- Usare UNION di LEFT JOIN e RIGHT JOIN
```

### CROSS JOIN (Prodotto Cartesiano)

Combina **ogni riga della tabella 1 con ogni riga della tabella 2**:

```sql
-- Produce 1000 righe se tabella1 ha 10 righe e tabella2 ha 100
SELECT * FROM tabella1 CROSS JOIN tabella2;
```

### Self Join

Unire una tabella con se stessa:

```sql
-- Trovare dipendenti con lo stesso manager
SELECT e1.nome, e1.manager_id, e2.nome as manager
FROM impiegato e1
JOIN impiegato e2 ON e1.manager_id = e2.id;
```