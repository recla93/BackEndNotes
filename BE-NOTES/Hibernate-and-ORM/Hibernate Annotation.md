### @Entity

Marca una classe come **entità persistente** (mapperà a una tabella):

```java
@Entity
@Table(name = "persona")  // Nome della tabella
public class Persona {
    // ...
}
```

### @Id e @GeneratedValue

```java
@Entity
public class Persona {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;  // Chiave primaria, auto-increment
}
```

**Strategie di generazione**:
- `IDENTITY`: delega al DB (AUTO_INCREMENT in MySQL)
- `SEQUENCE`: usa una sequenza DB (PostgreSQL, Oracle)
- `TABLE`: usa una tabella per tracciare i valori
- `AUTO`: Hibernate sceglie automaticamente

### @Column

Specifica il mapping di una colonna:

```java
@Entity
public class Persona {
    @Column(name = "nome_persona", length = 100, nullable = false)
    private String nome;
    
    @Column(unique = true)
    private String email;
}
```

### @Transient

Proprietà che **non** viene salvata nel database:

```java
@Entity
public class Persona {
    @Transient
    private String campoTemporaneo;  // Non va nel DB
}
```

### Relazioni: @OneToMany e @ManyToOne

```java
// Tabella padre
@Entity
public class Regione {
    @Id
    private int id;
    
    @OneToMany(mappedBy = "regione")
    private List<Provincia> province;
}

// Tabella figlia
@Entity
public class Provincia {
    @Id
    private int id;
    
    @ManyToOne
    @JoinColumn(name = "id_regione")  // Chiave esterna
    private Regione regione;
}
```

### Relazioni: @ManyToMany

```java
@Entity
public class Docente {
    @ManyToMany
    @JoinTable(
        name = "docente_materia",  // Tabella associativa
        joinColumns = @JoinColumn(name = "id_docente"),
        inverseJoinColumns = @JoinColumn(name = "id_materia")
    )
    private List<Materia> materie;
}

@Entity
public class Materia {
    @ManyToMany(mappedBy = "materie")
    private List<Docente> docenti;
}
```

### @Transactional

Gestisce le transazioni (tutto o niente):

```java
@Service
public class PersonaService {
    @Transactional
    public void salvaPersone(List<Persona> persone) {
        for (Persona p : persone) {
            repository.save(p);
        }
        // Se una eccezione avviene, tutto viene annullato (rollback)
    }
}
```

### @DiscriminatorColumn (Single Table Inheritance)

Usa una colonna per distinguere i sottotipi:

```java
@Entity
@Table(name = "persona")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo")
@DiscriminatorValue("persona")
public class Persona {
    @Id
    private int id;
    private String nome;
}

@Entity
@DiscriminatorValue("studente")
public class Studente extends Persona {
    private int numeroMatricola;
}

@Entity
@DiscriminatorValue("docente")
public class Docente extends Persona {
    private String stipendio;
}
```
