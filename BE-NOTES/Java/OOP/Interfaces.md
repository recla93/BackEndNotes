### Cos'è un'Interfaccia?

Un'**interfaccia** è un "contratto" che definisce **COSA** una classe deve fare (metodi astratti), ma non **COME** farlo:

```java
public interface Veicolo {
    void accelera();
    void frena();
    void gira();
}
```

### Implementazione di un'Interfaccia

```java
public class Auto implements Veicolo {
    @Override
    public void accelera() {
        System.out.println("Auto accelera");
    }
    
    @Override
    public void frena() {
        System.out.println("Auto frena");
    }
    
    @Override
    public void gira() {
        System.out.println("Auto gira");
    }
}
```

### Caratteristiche Importanti

- Una classe può implementare **infinite interfacce**
- Una classe può estendere **solo una classe**
- Le interfacce sono **apolidi**: non possono avere proprietà di istanza, solo costanti static final
- Tutti i metodi sono implicitamente `public abstract`

```java
// Una classe può implementare multiple interfacce
public class Auto implements Veicolo, Manutenibile, Assicurabile {
    // Implementare metodi di tutte e tre le interfacce
}
```

### Vantaggi delle Interfacce

1. **Decoupling**: separazione tra il "cosa fare" e il "come farlo"
2. **Polimorfismo**: una variabile di tipo interfaccia può riferirsi a diversi oggetti
3. **Contratto**: garantisce che una classe implementi determinati comportamenti
4. **Flessibilità**: facile cambiare implementazione

```java
// Questo metodo funziona con QUALSIASI implementazione di Veicolo
public void guidareMacchina(Veicolo v) {
    v.accelera();
    v.frena();
    v.gira();
}

// Uso
guidareMacchina(new Auto());      // Auto
guidareMacchina(new Moto());      // Moto
guidareMacchina(new Bicicletta()); // Bicicletta
```

### Interfaccia Funzionale

Un'interfaccia con **un solo metodo astratto**:

```java
@FunctionalInterface
public interface Condizione {
    boolean test(int x);
}

// Può essere usata con Lambda
Condizione c = x -> x > 10;  // Sintassi lambda
System.out.println(c.test(15));  // true
```

### Metodi Default nelle Interfacce (Java 8+)

```java
public interface Veicolo {
    void accelera();
    
    // Metodo default con implementazione
    default void suona() {
        System.out.println("Beep beep!");
    }
}

// La classe che implementa NON deve implementare suona()
public class Auto implements Veicolo {
    @Override
    public void accelera() {
        // ...
    }
    
    // suona() è ereditato dall'interfaccia
}
```