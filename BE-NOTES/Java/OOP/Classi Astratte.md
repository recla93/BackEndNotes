### Cos'è una Classe Astratta?

Una **classe astratta** è una classe che **non può essere istanziata** direttamente. Serve come punto di partenza per altre classi che la estendono:

```java
// ❌ Non si può istanziare
// Persona p = new Persona();  // ERRORE!

// ✅ Si estende
public class Studente extends Persona { }
Studente s = new Studente();  // OK
```

### Metodi Astratti

Un **metodo astratto** non ha implementazione, solo la firma:

```java
public abstract class Persona {
    // Metodo astratto - senza corpo
    public abstract boolean isValid();
    
    // Metodo concreto
    public void saluta() {
        System.out.println("Ciao");
    }
}

// La sottoclasse DEVE implementare il metodo astratto
public class Studente extends Persona {
    @Override
    public boolean isValid() {
        // Implementazione specifica per Studente
        return numeroMatricola > 0;
    }
}
```

### Cosa Contiene una Classe Astratta

- ✓ Proprietà di oggetto e di classe (static)
- ✓ Metodi statici e concreti
- ✓ Metodi astratti
- ✓ Costruttori
- ✗ Non può essere istanziata direttamente

```java
public abstract class Persona {
    private String nome;  // Proprietà di oggetto
    private static int contatore = 0;  // Proprietà di classe
    
    public Persona() {}  // Costruttore
    
    public static int getContatore() {  // Metodo statico
        return contatore;
    }
    
    public void saluta() {  // Metodo concreto
        System.out.println("Ciao");
    }
    
    public abstract void descrivi();  // Metodo astratto
}
```

### Classe Astratta vs Interfaccia

| Aspetto       | Classe Astratta     | Interfaccia           |
| ------------- | ------------------- | --------------------- |
| Istanziazione | No                  | No                    |
| Estensione    | Una sola            | Infinite              |
| Stato         | Sì (proprietà)      | No (solo costanti)    |
| Metodi        | Concreti e astratti | Astratti (di default) |
| Uso           | "è un" (IS-A)       | "può fare" (CAN-DO)   |
| Costruttore   | Sì                  | No                    |

---

## Classi Astratte

**Non può essere istanziata** nel main, viene usata come punto di partenza per altre classi, nasce per essere estesa.

​

## Cosa può contenere una classe astratta

- FINAL STATIC: Sì
    

- ​
    
- STATIC: Sì
    
- ​
    
- Metodi STATIC: Sì
    
- ​
    
- Proprietà di oggetto: Sì
    
- ​
    
- Metodi normali: Sì
    
- ​
    
- Metodi astratti: Sì
    

- ​
    

Una classe astratta può avere **TUTTO**.

​

## Caratteristiche principali

- È utilizzata come punto di partenza per altre classi
    

- ​
    
- È progettata per essere estesa da altre classi
    
- ​
    
- Può contenere proprietà e metodi statici
    
- ​
    
- Fornisce una struttura comune permettendo la definizione di metodi astratti
    

- ​
    

## Metodi Astratti

Metodo **senza corpo**, senza codice, solo firma. Verrà implementato ed utilizzato da altre classi.

​

Implica che ogni figlio **dovrà avere** questo metodo, ma le regole vengono definite dai figli.

​

Quando una classe concreta eredita da una classe astratta, è **obbligata a fare override** e implementare tutti i metodi astratti ereditati.

​