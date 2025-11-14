### Operatori di Confronto

| Operatore | Significato | Esempio |
|-----------|------------|---------|
| `=` | Uguale | `eta = 25` |
| `!=` oppure `<>` | Diverso | `eta != 25` |
| `>` | Maggiore | `eta > 25` |
| `<` | Minore | `eta < 25` |
| `>=` | Maggiore o uguale | `eta >= 25` |
| `<=` | Minore o uguale | `eta <= 25` |

### LIKE per Ricerche (Pattern Matching)

```sql
-- Contiene "Mario"
WHERE nome LIKE '%Mario%'

-- Inizia con "Mar"
WHERE nome LIKE 'Mar%'

-- Finisce con "io"
WHERE nome LIKE '%io'

-- Esattamente 4 caratteri
WHERE nome LIKE '____'
```

### Funzioni di Aggregazione

```sql
-- Contare righe
SELECT COUNT(*) FROM persona;

-- Contare valori non NULL
SELECT COUNT(nome) FROM persona;

-- Somma
SELECT SUM(eta) FROM persona;

-- Media
SELECT AVG(eta) FROM persona;

-- Massimo
SELECT MAX(eta) FROM persona;

-- Minimo
SELECT MIN(eta) FROM persona;
```

### ORDER BY

```sql
-- Ordinamento ascendente (default)
SELECT * FROM persona ORDER BY eta ASC;

-- Ordinamento discendente
SELECT * FROM persona ORDER BY eta DESC;

-- Più colonne
SELECT * FROM persona ORDER BY cognome ASC, nome ASC;
```

### GROUP BY

```sql
-- Raggruppare e contare
SELECT colore, COUNT(*) as quantita
FROM vestito
GROUP BY colore;

-- Con aggregazioni
SELECT regione, SUM(abitanti), AVG(abitanti)
FROM provincia
GROUP BY regione;

-- HAVING (filtro su aggregazioni)
SELECT colore, COUNT(*) as quantita
FROM vestito
GROUP BY colore
HAVING COUNT(*) > 5;
```

### LIMIT e OFFSET

```sql
-- Primi 10 risultati
SELECT * FROM persona LIMIT 10;

-- 10 risultati a partire dal 20°
SELECT * FROM persona LIMIT 10 OFFSET 20;

-- Equivalente (alcuni database)
SELECT * FROM persona LIMIT 20, 10;
```
