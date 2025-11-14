### Cos'è Spring Data JPA?

**Spring Data JPA** semplifica l'accesso ai dati tramite JPA/Hibernate. Invece di scrivere query manualmente, crei un **Repository**:

```java
// ❌ SENZA Spring Data JPA: devi scrivere tutto manualmente
@Service
public class PersonaService {
    @Autowired
    private SessionFactory sessionFactory;
    
    public List<Persona> findAll() {
        Session session = sessionFactory.openSession();
        try {
            return session.createQuery("FROM Persona", Persona.class).list();
        } finally {
            session.close();
        }
    }
}

// ✅ CON Spring Data JPA: il framework fa tutto
@Service
public class PersonaService {
    @Autowired
    private PersonaRepository repository;
    
    public List<Persona> findAll() {
        return repository.findAll();  // Semplice!
    }
}
```

### Interfaccia Repository

```java
// Spring Data JPA fornisce automaticamente CRUD
public interface PersonaRepository extends JpaRepository<Persona, Integer> {
    // JpaRepository<Entity, IdType>
    // Persona = entity class
    // Integer = tipo della chiave primaria
}
```

### Metodi Automatici di JpaRepository

```java
// CREATE
Persona p = new Persona();
p.setNome("Mario");
repository.save(p);

// READ
Persona p = repository.findById(1).orElse(null);  // Optional
Persona p = repository.getById(1);  // Direct (deprecated in Java 18+)
List<Persona> all = repository.findAll();

// UPDATE
p.setEta(31);
repository.save(p);  // save serve sia per insert che update

// DELETE
repository.deleteById(1);
repository.delete(p);
repository.deleteAll();

// QUERY
boolean exists = repository.existsById(1);
long count = repository.count();
```

### Query Personalizzate

**Naming Convention** - Spring Data JPA capisce il nome del metodo:

```java
public interface PersonaRepository extends JpaRepository<Persona, Integer> {
    // Spring interpreta il nome e genera la query
    
    // Find all = SELECT *
    List<Persona> findAll();
    
    // Find by + PropertyName = WHERE property
    List<Persona> findByNome(String nome);
    List<Persona> findByEta(int eta);
    
    // GreaterThan, LessThan, Between, etc.
    List<Persona> findByEtaGreaterThan(int eta);
    List<Persona> findByEtaLessThan(int eta);
    List<Persona> findByEtaBetween(int minEta, int maxEta);
    
    // And, Or
    List<Persona> findByNomeAndEta(String nome, int eta);
    List<Persona> findByNomeOrCognome(String nome, String cognome);
    
    // Like (CONTAINS)
    List<Persona> findByNomeContaining(String parte);
    
    // OrderBy
    List<Persona> findByEtaOrderByNomeAsc(int eta);
    
    // Distinct
    List<Persona> findDistinctByCognome(String cognome);
    
    // First, Top (LIMIT)
    Persona findFirstByOrderByEtaDesc();
    List<Persona> findTop10ByOrderByEtaDesc();
}

// Uso
List<Persona> maggiorenni = repository.findByEtaGreaterThanOrEqual(18);
List<Persona> mario = repository.findByNome("Mario");
List<Persona> dicembre = repository.findByNomeContaining("Mario");
```

### @Query Personalizzate

Per query complesse, usa `@Query`:

```java
public interface PersonaRepository extends JpaRepository<Persona, Integer> {
    
    // Query JPQL
    @Query("SELECT p FROM Persona p WHERE p.eta > :minEta ORDER BY p.nome")
    List<Persona> findAdulti(@Param("minEta") int minEta);
    
    // With LIKE
    @Query("SELECT p FROM Persona p WHERE p.nome LIKE %:nome%")
    List<Persona> search(@Param("nome") String nome);
    
    // Aggregazione
    @Query("SELECT COUNT(p) FROM Persona p WHERE p.eta > :minEta")
    long countAdulti(@Param("minEta") int minEta);
    
    // Native SQL (last resort)
    @Query(value = "SELECT * FROM persona WHERE eta > ?1", nativeQuery = true)
    List<Persona> findAdultiNativeSQL(int minEta);
}
```