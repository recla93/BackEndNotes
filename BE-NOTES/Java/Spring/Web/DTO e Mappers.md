### Cos'è un DTO?

Un **DTO** (Data Transfer Object) è un oggetto che contiene **solo i dati da trasferire**, senza logica. Separa le **entità del DB** dai **dati inviati al client**:

```
Entity (DB)      DTO (API)       JSON
┌─────────┐      ┌────────┐      ┌──────┐
│ Persona │  →   │Persona │  →   │{"id" │
│ - id    │      │DTO     │      │: 1,  │
│ - nome  │      │ - id   │      │"nome"│
│ - eta   │      │ - nome │      │:...  │
│ - pwd   │      │ - eta  │      └──────┘
│ - email │      └────────┘      (solo dati pubblici)
│ - token │
└─────────┘
(contiene
dati sensibili)
```

### Esempio di DTO

```java
// Entity (in DB)
@Entity
public class Persona {
    @Id
    private int id;
    private String nome;
    private int eta;
    private String password;      // ✗ Sensibile
    private String email;
    private String token;          // ✗ Sensibile
}

// DTO (inviato al client)
public class PersonaDTO {
    private int id;
    private String nome;
    private int eta;              // ✓ Pubblico
    
    // Constructor, getter, setter
    public PersonaDTO(int id, String nome, int eta) {
        this.id = id;
        this.nome = nome;
        this.eta = eta;
    }
}

// Uso in Controller
@RestController
public class PersonaController {
    @Autowired
    private PersonaService service;
    @Autowired
    private PersonaMapper mapper;
    
    @GetMapping("/persone/{id}")
    public PersonaDTO getOne(@PathVariable int id) {
        Persona p = service.findById(id).orElseThrow();
        return mapper.toDTO(p);  // Converti Entity a DTO
    }
}
```

### Mapper

Un **mapper** converte Entity ↔ DTO:

```java
// Interfaccia mapper
@Mapper(componentModel = "spring")
public interface PersonaMapper {
    PersonaDTO toDTO(Persona persona);
    Persona toEntity(PersonaDTO dto);
    
    List<PersonaDTO> toDTOList(List<Persona> persone);
    List<Persona> toEntityList(List<PersonaDTO> dtos);
}

// Con MapStruct (libreria)
// Aggiungere dipendenza: org.mapstruct:mapstruct
// L'implementazione viene generata automaticamente

// Uso
@Service
public class PersonaService {
    @Autowired
    private PersonaRepository repository;
    @Autowired
    private PersonaMapper mapper;
    
    public PersonaDTO getPersonaDTO(int id) {
        Persona p = repository.findById(id).orElseThrow();
        return mapper.toDTO(p);  // Conversione automatica
    }
}
```

### Vantaggi dei DTO

- ✓ **Separazione di responsabilità**: Entity vs API
- ✓ **Sicurezza**: nascondi dati sensibili
- ✓ **Flessibilità**: cambia DTO senza cambiare Entity
- ✓ **Performance**: invia solo dati necessari
- ✓ **Versioning**: diversi DTO per diverse versioni API
