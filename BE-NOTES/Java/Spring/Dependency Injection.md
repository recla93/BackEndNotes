**Dependency Injection** significa che Spring **fornisce** le dipendenze che una classe ha bisogno:

```java
// ❌ MALE: Coupling stretto
public class PersonaController {
    private PersonaService service = new PersonaService();  // Crea manualmente
}

// ✅ BENE: Spring inietta
@RestController
public class PersonaController {
    @Autowired
    private PersonaService service;  // Spring lo fornisce
}
```

**Vantaggi**:
- ✓ Riduce l'accoppiamento (coupling)
- ✓ Facilita i test (puoi mockare le dipendenze)
- ✓ Codice più pulito
- ✓ Facile cambiare implementazione

### Tipi di Dependency Injection

**1. Constructor Injection** (consigliato):
```java
@Service
public class PersonaService {
    private PersonaRepository repository;
    
    // Spring inietta tramite costruttore
    public PersonaService(PersonaRepository repository) {
        this.repository = repository;
    }
}
```

**2. Setter Injection**:
```java
@Service
public class PersonaService {
    private PersonaRepository repository;
    
    @Autowired
    public void setRepository(PersonaRepository repository) {
        this.repository = repository;
    }
}
```

**3. Field Injection** (sconsigliato):
```java
@Service
public class PersonaService {
    @Autowired
    private PersonaRepository repository;  // Diretto
}
```

