## Service Layer

### Cos'è il Service Layer?

Il **Service Layer** contiene la **logica di business**. Separa la logica da controlloriI e repository:

```
Controller (riceve richiesta)
    ↓
Service (contiene logica)
    ↓
Repository (accesso dati)
    ↓
Database
```

### Esempio di Service

```java
@Service
public class PersonaService {
    @Autowired
    private PersonaRepository repository;
    
    // Logica di business: validazione
    public Persona salvaPersona(Persona p) {
        if (p.getNome() == null || p.getNome().isBlank()) {
            throw new IllegalArgumentException("Nome richiesto");
        }
        if (p.getEta() < 0 || p.getEta() > 150) {
            throw new IllegalArgumentException("Età non valida");
        }
        return repository.save(p);
    }
    
    // Logica di business: calcoli
    public List<Persona> getMaggiorenni() {
        List<Persona> tutti = repository.findAll();
        return tutti.stream()
            .filter(p -> p.getEta() >= 18)
            .collect(Collectors.toList());
    }
    
    // Logica di business: transazioni
    @Transactional
    public void trasferisciDati(int idSorgente, int idDestinazione) {
        Persona sorgente = repository.findById(idSorgente).orElseThrow();
        Persona destinazione = repository.findById(idDestinazione).orElseThrow();
        
        destinazione.setEta(sorgente.getEta());
        repository.save(destinazione);
        // Se eccezione: rollback automatico
    }
}
```

### Vantaggi del Service Layer

- ✓ Separa logica da controllori
- ✓ Logica riutilizzabile
- ✓ Facile da testare
- ✓ Gestione transazioni centralizzata
- ✓ Gestione eccezioni
