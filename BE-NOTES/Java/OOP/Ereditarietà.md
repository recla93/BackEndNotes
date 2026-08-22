---
topic: "Ereditarietà"
nav_prev: "[[Override, interfaces, Overload.md]]"
nav_next: "[[Classi Astratte.md]]"
---
### Concetto Fondamentale

L'**ereditarietÃ ** permette a una classe di ereditare proprietÃ  e metodi da un'altra classe (superclasse):

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

### Modificatori di VisibilitÃ 

| Modificatore | Classe | Package | Sottoclasse | Ovunque |
|--------------|--------|---------|------------|---------|
| `public` | âœ“ | âœ“ | âœ“ | âœ“ |
| `protected` | âœ“ | âœ“ | âœ“ | âœ— |
| `default` | âœ“ | âœ“ | âœ— | âœ— |
| `private` | âœ“ | âœ— | âœ— | âœ— |

**Protected** Ã¨ il modificatore ideale per le proprietÃ  di superclassi che si vogliono ereditare:

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

L'ereditarietÃ  aiuta a evitare la ripetizione di codice:

```java
// âŒ MALE - Ripetizione
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

// âœ… BENE - EreditarietÃ 
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

## Ereditarietà e metodi astratti
Quando una superclasse astratta o un'interfaccia dichiara metodi astratti (senza corpo), le sottoclassi **devono** fornire un'implementazione concreta tramite override. È il meccanismo che garantisce che ogni sottoclasse abbia il comportamento specifico, pur rispettando il contratto definito dalla superclasse.

Vedi [[Classi Astratte]] per i dettagli — l'ereditarietà è il presupposto strutturale, le classi astratte sono il veicolo che impone il contratto.

## Ereditarietà e polimorfismo

L'override dei metodi è la base del polimorfismo in Java: una variabile di tipo superclasse può riferirsi a oggetti di sottoclassi diverse, e la chiamata a un metodo override esegue l'implementazione della **classe concreta** (non di quella dichiarata).

```java
Persona p1 = new Studente();
Persona p2 = new Docente();
p1.saluta();  // esegue Studente.saluta() — non Persona.saluta()
p2.saluta();  // esegue Docente.saluta() — comportamento diverso
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Ereditare per riuso invece che per specializzazione | Gerarchia innaturale, difficile da mantenere | "Voglio riusare quei metodi, quindi estendo" | Preferisci composizione (has-a) a ereditarietà (is-a) |
| Downcasting senza `instanceof` | `ClassCastException` a runtime | Si assume che un oggetto sia di un tipo specifico senza verificare | Controlla sempre con `instanceof` prima del downcast |
| Override senza `@Override` | Non ti accorgi che la firma è sbagliata (fai overload invece) | Il metodo non matcha nessun metodo del genitore, ma compila lo stesso | Aggiungi `@Override` — il compilatore verifica la firma |
| Chiamare `super` quando non serve | Comportamento duplicato o errato | Chiami `super.metodo()` anche quando vuoi sostituire completamente | Se l'override è totale, non chiamare `super` |
| Metodo privato "ereditato" | Non compila: il figlio non vede il metodo privato del genitore | I metodi `private` non sono visibili alle sottoclassi | Usa `protected` se il figlio deve accedere |
| Deep inheritance (> 3 livelli) | Manutenzione impossibile, cambiamenti a cascata | Una modifica alla radice impatta tutte le sottoclassi | Preferisci composizione o interfacce. Massimo 2-3 livelli |

Vedi [[Polimorfismo]] per il trattamento completo.
