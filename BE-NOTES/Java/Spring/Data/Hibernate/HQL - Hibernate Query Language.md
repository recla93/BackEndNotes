---
topic: "HQL — Hibernate Query Language"
nav_prev: "[[Relazioni e Mappature.md]]"
nav_next: "[[../Lock Ottimistico e Pessimistico.md]]"
---
### Cos'è HQL?

**HQL** è SQL **orientato a oggetti**. Scrivi query pensando agli oggetti, non alle tabelle:

```
SQL (orientato a tabelle):
SELECT * FROM persona WHERE eta > 25;

HQL (orientato a oggetti):
SELECT p FROM Persona p WHERE p.eta > 25;
```

### SELECT

```java
// Tutte le persone
Query q = session.createQuery("FROM Persona");
List<Persona> persone = q.list();

// Con parametri
Query q = session.createQuery("FROM Persona WHERE eta > :minEta");
q.setParameter("minEta", 18);
List<Persona> persone = q.list();

// Solo nomi
Query q = session.createQuery("SELECT p.nome FROM Persona p");
List<String> nomi = q.list();

// Creazione di oggetti DTO
Query q = session.createQuery(
    "SELECT new PersonaDTO(p.nome, p.eta) FROM Persona p"
);
```

### WHERE

```java
// Condizione singola
Query q = session.createQuery("FROM Persona WHERE eta >= :minEta");
q.setParameter("minEta", 18);

// Condizioni multiple
Query q = session.createQuery(
    "FROM Persona WHERE eta >= :minEta AND cognome LIKE :cognome"
);
q.setParameter("minEta", 18);
q.setParameter("cognome", "%Rossi%");
```

### JOIN

```java
// Inner Join
Query q = session.createQuery(
    "SELECT p FROM Provincia p " +
    "JOIN p.regione r " +
    "WHERE r.nome = 'Lazio'"
);

// Left Join (per includere province senza regione)
Query q = session.createQuery(
    "SELECT p FROM Provincia p " +
    "LEFT JOIN p.regione r"
);

// With ON
Query q = session.createQuery(
    "FROM Provincia p " +
    "LEFT JOIN Regione r ON p.regione.id = r.id"
);
```

### ORDER BY

```java
Query q = session.createQuery(
    "FROM Persona ORDER BY eta DESC"
);
List<Persona> persone = q.list();
```

### Aggregazioni

```java
// COUNT
Query q = session.createQuery("SELECT COUNT(*) FROM Persona");
Long count = (Long) q.uniqueResult();

// AVG
Query q = session.createQuery("SELECT AVG(p.eta) FROM Persona p");
Double media = (Double) q.uniqueResult();

// MAX
Query q = session.createQuery("SELECT MAX(p.eta) FROM Persona p");
Integer massimo = (Integer) q.uniqueResult();

// GROUP BY
Query q = session.createQuery(
    "SELECT p.cognome, COUNT(*) FROM Persona p " +
    "GROUP BY p.cognome"
);
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Usare nomi di tabella SQL invece di nomi di entità | `QueryException: "Persona is not mapped"` | HQL usa i nomi delle classi Java, non i nomi delle tabelle | Scrivi `FROM Persona` non `FROM persona` |
| Parametro non nominato correttamente | `Parameter value not set` | Usi `:nome` nel JPQL ma `setParameter(0, val)` posizionale | Sii consistente: sempre parametri nominativi `:nome` con `setParameter("nome", val)` |
| Confondere `JOIN` HQL con SQL JOIN | Sintassi errata o result set inatteso | In HQL fai JOIN sulla PROPRIETÀ, non sulla tabella | `p.regione` non `p INNER JOIN regione` — usa l'associazione Java |
| `SELECT *` non funziona in HQL | Errore di sintassi | `*` non è valido in HQL | Usa `FROM Persona` (senza SELECT) o `SELECT p FROM Persona p` |
| Query nativa usata dove basta JPQL | Accoppiamento al DB, query non portabile | "Tanto è uguale" | Preferisci JPQL/HQL per query standard; native solo per feature DB-specifiche |

### Named Query

```java
@Entity
@NamedQueries({
    @NamedQuery(
        name = "Persona.findByEta",
        query = "SELECT p FROM Persona p WHERE p.eta = :eta"
    ),
    @NamedQuery(
        name = "Persona.findAll",
        query = "SELECT p FROM Persona p ORDER BY p.nome"
    )
})
public class Persona {
    // ...
}

// Uso
Query q = session.getNamedQuery("Persona.findByEta");
q.setParameter("eta", 25);
List<Persona> persone = q.list();
```