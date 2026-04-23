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

ORM == Far sapere a java la relazione tra Classi e DB (facciamo sapere a Java com’è fatto il nostro DB)  
JAKARTA PERSISTENCE -> ORM  

ORM sta per Object-Relational Mapping.
È una metodologia che permette di mappare gli oggetti delle classi Java alle tabelle di un database relazionale. In pratica, crea una corrispondenza tra le entità Java e la struttura del database, semplificando la gestione dei dati tra l'applicazione e il database stesso. Non è un framework di per sé, ma una tecnica implementata da vari strumenti come JPA in Spring.  
  
** Repository= PARTE DEL PROGRAMMA CHE FA CRUD PER UNA DETERMINATA ENTITA’** (Perché lo fa per un’entità per volta)

  
Una repository  è un pattern che fornisce un'astrazione del livello di accesso ai dati. Essenzialmente, funge da intermediario tra la logica di business dell'applicazione e il database, permettendo di separare la logica di accesso ai dati dal resto del codice. Questo pattern facilita le operazioni CRUD (Create, Read, Update, Delete) sul database, nascondendo la complessità delle query e delle operazioni di persistenza dei dati.