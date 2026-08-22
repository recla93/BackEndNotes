---
topic: "Classi Astratte"
nav_prev: "[[Ereditarietà.md]]"
nav_next: "[[Interfaces.md]]"
---
### Cos'Ã¨ una Classe Astratta?

Una **classe astratta** Ã¨ una classe che **non puÃ² essere istanziata** direttamente. Serve come punto di partenza per altre classi che la estendono:

```java
// âŒ Non si puÃ² istanziare
// Persona p = new Persona();  // ERRORE!

// âœ… Si estende
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

- âœ“ ProprietÃ  di oggetto e di classe (static)
- âœ“ Metodi statici e concreti
- âœ“ Metodi astratti
- âœ“ Costruttori
- âœ— Non puÃ² essere istanziata direttamente

```java
public abstract class Persona {
    private String nome;  // ProprietÃ  di oggetto
    private static int contatore = 0;  // ProprietÃ  di classe
    
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
| Stato         | SÃ¬ (proprietÃ )      | No (solo costanti)    |
| Metodi        | Concreti e astratti | Astratti (di default) |
| Uso           | "Ã¨ un" (IS-A)       | "puÃ² fare" (CAN-DO)   |
| Costruttore   | SÃ¬                  | No                    |

---

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Istanziare una classe astratta | Errore di compilazione | Le classi astratte non possono essere istanziate | Correggi l'istanza o rendi la classe concreta rimuovendo `abstract` |
| Dimenticare di implementare un metodo astratto | Errore di compilazione nella sottoclasse concreta | La classe figlia non implementa tutti i metodi astratti del genitore | Implementa tutti i metodi astratti, oppure dichiara la figlia `abstract` |
| Astratto senza metodo astratto | Classe astratta con solo metodi concreti — possibile ma inutile | Serve una base comune ma nessun comportamento da forzare | Valuta se serve una classe astratta o se una classe concreta basta |
| Usare classe astratta dove basta un'interfaccia | Accoppiamento stretto (ereditarietà singola bruciata) | "Classe astratta con solo metodi astratti" = interfaccia | Usa `interface` invece di `abstract class` se non c'è stato condiviso |
