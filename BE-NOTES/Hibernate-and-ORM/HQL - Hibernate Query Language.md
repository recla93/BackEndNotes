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