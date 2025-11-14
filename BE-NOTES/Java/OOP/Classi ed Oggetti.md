### Concetto Fondamentale

Una **classe** è un modello (template) che definisce:
- **Proprietà** (state): le caratteristiche dell'oggetto
- **Metodi** (behavior): le azioni che l'oggetto può compiere

```java
public class Persona {
    // PROPRIETÀ (state)
    private String nome;
    private int eta;
    
    // METODI (behavior)
    public void saluta() {
        System.out.println("Ciao, sono " + nome);
    }
}
```

### Creazione di Oggetti

```java
// Istanziazione: creazione di un'istanza della classe
Persona mario = new Persona();  // Nuovo oggetto Persona

// Accesso alle proprietà tramite getter/setter
mario.setNome("Mario");
mario.setEta(30);
```

### Tipo Concreto vs Tipo Formale

Questo è un concetto **fondamentale** in Java:

```java
// Tipo formale: il tipo della variabile (COSA PUÒ FARE)
// Tipo concreto: l'oggetto effettivo in memoria (COME FUNZIONA)

Studente mario = new Studente();
// Tipo concreto: Studente
// Tipo formale: Studente

Persona p = mario;
// Tipo concreto: Studente (NON CAMBIA!)
// Tipo formale: Persona (cambiato dalla variabile)
```

**Importanza**: Il tipo formale determina **quali metodi puoi chiamare**, mentre il tipo concreto determina **come viene effettivamente eseguito**.

```java
Persona p = new Studente();

// Posso chiamare solo metodi di Persona
p.saluta();  // OK

// NON posso chiamare metodi specifici di Studente
// p.prendi Voto();  // ERRORE - Persona non ha questo metodo

// Devo fare downcasting
if (p instanceof Studente) {
    Studente s = (Studente) p;  // Downcasting
    s.prendiVoto(8);  // Ora OK
}
```

### This

L'operatore `this` rappresenta **l'oggetto corrente** su cui viene chiamato il metodo:

```java
public class Persona {
    private String nome;
    
    public void setNome(String nome) {
        this.nome = nome;  // this.nome = proprietà dell'oggetto
                           // nome = parametro del metodo
    }
}
```

### Incapsulamento (Encapsulation)

L'incapsulamento nasconde i dettagli interni dell'oggetto:

```java
public class Persona {
    // Proprietà private: nascoste dall'esterno
    private String nome;
    private int eta;
    
    // Getter: lettura controllata
    public String getNome() {
        return nome;
    }
    
    // Setter: modifica controllata con validazione
    public void setNome(String nome) {
        if (nome != null && !nome.isBlank()) {
            this.nome = nome;
        }
    }
}
```

**Vantaggi**:
- Validazione dei dati
- Nascondimento dell'implementazione
- Facilità di manutenzione
- Riduzione dell'accoppiamento

### Stato dell'Oggetto

Lo **stato** è l'insieme dei valori delle proprietà in un momento specifico:

```java
Persona p = new Persona();
p.setNome("Mario");
p.setEta(30);
// Stato di p: {nome: "Mario", eta: 30}

p.setEta(31);
// Nuovo stato di p: {nome: "Mario", eta: 31}
```