---
topic: "ORM"
nav_prev: "[[../Entity Mapping.md]]"
nav_next: "[[JPA - Java Persistence API.md]]"
---
### Cos'è ORM?

**ORM** (Object-Relational Mapping) è una tecnica che **mappa gli oggetti Java alle righe delle tabelle del database**.

```
Oggetto Java              Riga nel Database
┌─────────────────┐      ┌─────────────────┐
│ Persona         │      │ persona table   │
│ - nome: String  │ ←→   │ - nome: VARCHAR │
│ - eta: int      │      │ - eta: INT      │
└─────────────────┘      └─────────────────┘
```

### Problema Senza ORM (JDBC Manuale)

```java
// Prima di ORM: dovevi convertire manualmente
ResultSet rs = statement.executeQuery("SELECT * FROM persona");
while (rs.next()) {
    Persona p = new Persona();
    p.setNome(rs.getString("nome"));
    p.setEta(rs.getInt("eta"));
    persone.add(p);
}
```

### Soluzione Con ORM (Hibernate)

```java
// Con ORM: conversione automatica
List<Persona> persone = session.createQuery(
    "FROM Persona", 
    Persona.class
).list();  // Semplice!
```

### Vantaggi di ORM

- ✓ Riduce il codice SQL manuale
- ✓ Migliore astrazione dei dati
- ✓ Facile sincronizzazione tra Java e DB
- ✓ Relazioni automaticamente gestite
- ✓ Sicurezza (protezione da SQL injection)

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Mappare ogni tabella a ogni costo | Over-engineering per viste e query semplici | "Ogni risultato SQL deve essere un'entità" | Usa `@SqlResultSetMapping`, DTO o proiezioni per query di sola lettura |
| Ignorare i proxy Hibernate | `LazyInitializationException` fuori dalla transazione | Si accede a una relazione lazy quando la sessione è chiusa | Carica i dati necessari dentro la transazione con `JOIN FETCH` o `@EntityGraph` |
| N+1 query | Performance pessima su liste | Si itera su entità e per ogni una lazy load | Usa `JOIN FETCH`, `@EntityGraph`, o `@BatchSize` |
| Modificare entità fuori dalla transazione | Le modifiche NON vengono salvate | Hibernate traccia i cambiamenti solo dentro una transazione attiva | Fai tutte le modifiche dentro il metodo `@Transactional` |