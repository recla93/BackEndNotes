### Concetto Fondamentale

L'**ereditarietà** permette a una classe di ereditare proprietà e metodi da un'altra classe (superclasse):

```java
// Superclasse
public class Persona {
    protected String nome;  // protected = accessibile da sottoclassi
    protected int eta;
    
    public void saluta() {
        System.out.println("Ciao, sono " + nome);
    }
}

// Sottoclasse
public class Studente extends Persona {
    private int numeroMatricola;
    
    // Eredita nome, eta, e il metodo saluta()
    // Aggiunge numero di matricola
}
```

### Modificatori di Visibilità

| Modificatore | Classe | Package | Sottoclasse | Ovunque |
|--------------|--------|---------|------------|---------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| `default` | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

**Protected** è il modificatore ideale per le proprietà di superclassi che si vogliono ereditare:

```java
public class Persona {
    // I sottotipi potranno accedere direttamente a nome
    protected String nome;
    private int eta;  // I sottotipi NON potranno accedere
}
```

### Upcasting e Downcasting

**Upcasting**: cambiare il tipo formale verso l'alto (verso il supertipo):

```java
Studente mario = new Studente();
Persona p = mario;  // Upcasting (implicito, sempre sicuro)
```

**Downcasting**: cambiare il tipo formale verso il basso (verso il sottotipo):

```java
Persona p = new Studente();
Studente mario = (Studente) p;  // Downcasting (esplicito, potenzialmente rischioso)
```

**Verificare il tipo prima di downcasting**:

```java
if (p instanceof Studente) {
    Studente mario = (Studente) p;  // Sicuro
    mario.prendiVoto();
}
```

### DRY Principle (Don't Repeat Yourself)

L'ereditarietà aiuta a evitare la ripetizione di codice:

```java
// ❌ MALE - Ripetizione
public class Studente {
    private String nome;
    private int eta;
    private int numeroMatricola;
}

public class Docente {
    private String nome;
    private int eta;
    private String stipendio;
}

// ✅ BENE - Ereditarietà
public class Persona {
    protected String nome;
    protected int eta;
}

public class Studente extends Persona {
    private int numeroMatricola;
}

public class Docente extends Persona {
    private String stipendio;
}
```

### Super

La parola chiave `super` accede ai metodi della superclasse:

```java
public class Studente extends Persona {
    @Override
    public void saluta() {
        super.saluta();  // Chiama il metodo della superclasse
        System.out.println("Sono uno studente");
    }
}
```

### Principio di Liskov

Un sottotipo deve essere **sempre sostituibile** al suo supertipo:

```java
Persona p1 = new Studente();  // OK
Persona p2 = new Docente();   // OK
Persona p3 = new Persona();   // OK

// Tutti gli oggetti di tipo Persona (formale) possono essere
// studenti, docenti, o persone generiche (concreti)
```



/////

EREDITARIETA’ =

 redita dei MEMBRI / proprietà  / variabili in un'altra classe che sarà specializzata in altro.  
Es: Persona padre di Studente (persona che è Studente)  
vogliamo passare  
DRY: non ripetere le proprietà del SUPER nel sottotipo (Single Source of truth)

In Java, "DRY" è l'acronimo di "Don't Repeat Yourself". È un principio di programmazione che incoraggia la riduzione della duplicazione del codice. L'idea è di evitare di scrivere lo stesso codice o logica più volte, promuovendo invece la riusabilità e la manutenzione del codice. Applicando il principio DRY, il codice diventa più pulito, più facile da mantenere e meno soggetto a errori.

  
**Ovverride**: Vogliamo sovrascrivere il metodo di una funzione che viene ereditata.  
ESEMPIO: si fa per migliorarlo o per farne un altro utilizzo in base alle esigenze.  
  
**Implementazione di metodi astratti:** Quando una classe astratta o un'interfaccia definisce metodi astratti, le sottoclassi devono sovrascrivere questi metodi per fornire una concreta implementazione.  
Polimorfismo: L'override permette di sfruttare il polimorfismo, dove una superclasse può riferirsi a oggetti di sottoclassi diverse e chiamare i metodi sovrascritti, ottenendo comportamenti diversi a seconda dell'oggetto.