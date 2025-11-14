### Relazione OneToMany

**Un studente ha molte presenze**:

```java
@Entity
public class Studente {
    @Id
    private int id;
    private String nome;
    
    @OneToMany(mappedBy = "studente")  // Quale proprietà in Presenza
    private List<Presenza> presenze;
}

@Entity
public class Presenza {
    @Id
    private int id;
    private LocalDate data;
    
    @ManyToOne
    @JoinColumn(name = "id_studente")  // Chiave esterna
    private Studente studente;
}
```

### Single Source Of Truth (SSOT) nelle Relazioni

**Importante**: deve esserci un unico punto di verità per la relazione:

```java
// ✅ CORRETTO
Studente s = new Studente();
Presenza p = new Presenza();
p.setStudente(s);           // Imposto la relazione nel figlio
s.getPresenze().add(p);     // Aggiungo il figlio al padre

// Metodo helper nel padre (consigliato)
@Entity
public class Studente {
    @OneToMany(mappedBy = "studente")
    private List<Presenza> presenze = new ArrayList<>();
    
    public void addPresenza(Presenza p) {
        presenze.add(p);
        p.setStudente(this);  // Mantiene la sincronizzazione
    }
}

// Uso
Studente s = new Studente();
Presenza p = new Presenza();
s.addPresenza(p);  // Sincronizzazione automatica
```

### MappedBy

La proprietà `mappedBy` **evita la duplicazione**:

```java
// Sbagliato: @JoinColumn da entrambi i lati
@Entity
public class Studente {
    @OneToMany
    @JoinColumn(name = "id_studente")  // ❌ Non mettere qui
    private List<Presenza> presenze;
}

// Corretto: @JoinColumn solo nel figlio, @OneToMany con mappedBy nel padre
@Entity
public class Studente {
    @OneToMany(mappedBy = "studente")  // ✅ Dire "la relazione è mappata da studente in Presenza"
    private List<Presenza> presenze;
}

@Entity
public class Presenza {
    @ManyToOne
    @JoinColumn(name = "id_studente")  // ✅ La chiave esterna è qui
    private Studente studente;
}
```

### FetchType: EAGER vs LAZY

```java
// EAGER: carica subito
@ManyToOne(fetch = FetchType.EAGER)
private Regione regione;  // Quando leggi una Provincia, carica anche la Regione

// LAZY: carica solo se acceduto (default)
@ManyToOne(fetch = FetchType.LAZY)
private Regione regione;  // La Regione viene caricata solo quando chiami provincia.getRegione()
```

### Cascade Operations

Quando salvi il padre, salva automaticamente i figli:

```java
@Entity
public class Regione {
    @OneToMany(cascade = CascadeType.ALL)
    private List<Provincia> province;
}

// Salva anche le province
regione.addProvincia(provincia);
session.save(regione);  // Salva regione e province
```

### Tabelle con 2 Foreign Key (Joined Inheritance)

```java
// Tabella padre
@Entity
@Table(name = "persona")
public class Persona {
    @Id
    @GeneratedValue
    private int id;
    private String nome;
}

// Tabella figlio (Joined inheritance)
@Entity
@Table(name = "studente")
@PrimaryKeyJoinColumn(name = "id")  // Condivide l'ID con Persona
public class Studente extends Persona {
    @Column(name = "numero_matricola")
    private int numeroMatricola;
}

@Entity
@Table(name = "docente")
@PrimaryKeyJoinColumn(name = "id")  // Condivide l'ID con Persona
public class Docente extends Persona {
    @Column(name = "stipendio")
    private double stipendio;
}
```

**Schema del database**:
```sql
-- Tabella persona
CREATE TABLE persona (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100)
);

-- Tabella studente (ha FK che è anche PK)
CREATE TABLE studente (
    id INT PRIMARY KEY,
    numero_matricola INT,
    FOREIGN KEY (id) REFERENCES persona(id) ON DELETE CASCADE
);

-- Tabella docente (ha FK che è anche PK)
CREATE TABLE docente (
    id INT PRIMARY KEY,
    stipendio DECIMAL,
    FOREIGN KEY (id) REFERENCES persona(id) ON DELETE CASCADE
);
```

**Query di JOIN in Java**:
```java
// Recuperare uno studente
Studente s = session.get(Studente.class, 1);
// Hibernate farà il JOIN automaticamente tra persona e studente

// HQL
Query q = session.createQuery(
    "FROM Studente s WHERE s.numeroMatricola = :nm"
);
q.setParameter("nm", 12345);
List<Studente> studenti = q.list();
```

---