### Cos'è il Repository Pattern?

Il **Repository** astrae il livello di accesso ai dati. Separa la logica di business dall'accesso ai dati:

```
Controller → Service → Repository → Database
                        (astrazione)
```

### Interfaccia Repository Personalizzata

```java
// Interfaccia personalizzata
public interface PersonaRepository extends JpaRepository<Persona, Integer> {
    // Metodi automatici da JpaRepository
    
    // Metodi personalizzati
    List<Persona> findByEtaGreaterThan(int eta);
    List<Persona> findByNomeContaining(String nome);
    
    @Query("SELECT p FROM Persona p WHERE p.eta > :minEta")
    List<Persona> findAdulti(@Param("minEta") int minEta);
}

// Implementazione (Spring la crea automaticamente)
// Non devi scrivere niente! Spring genera l'implementazione

// Uso in Service
@Service
public class PersonaService {
    @Autowired
    private PersonaRepository repository;
    
    public List<Persona> getAdulti() {
        return repository.findAdulti(18);
    }
}
```

### Custom Repository Implementation

Per query molto complesse, puoi fornire implementazione personalizzata:

```java
// Interfaccia aggiuntiva
public interface PersonaRepositoryCustom {
    List<Persona> ricercaAvanzata(String nome, int minEta, int maxEta);
}

// Implementazione
@Repository
public class PersonaRepositoryImpl implements PersonaRepositoryCustom {
    @Autowired
    private EntityManager em;
    
    @Override
    public List<Persona> ricercaAvanzata(String nome, int minEta, int maxEta) {
        // Implementazione complessa con Criteria API
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Persona> cq = cb.createQuery(Persona.class);
        Root<Persona> root = cq.from(Persona.class);
        
        List<Predicate> predicates = new ArrayList<>();
        if (nome != null) {
            predicates.add(cb.like(root.get("nome"), "%" + nome + "%"));
        }
        predicates.add(cb.between(root.get("eta"), minEta, maxEta));
        
        cq.where(predicates.toArray(new Predicate[0]));
        return em.createQuery(cq).getResultList();
    }
}

// Repository che estende sia JpaRepository che PersonaRepositoryCustom
public interface PersonaRepository 
    extends JpaRepository<Persona, Integer>, PersonaRepositoryCustom {
}

// Uso
@Service
public class PersonaService {
    @Autowired
    private PersonaRepository repository;
    
    public List<Persona> ricerca() {
        return repository.ricercaAvanzata("Mario", 18, 30);
    }
}
```
