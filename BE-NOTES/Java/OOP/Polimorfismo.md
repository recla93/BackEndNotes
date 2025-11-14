### Definizione

Il **polimorfismo** significa "molte forme". In Java:

1. **Polimorfismo di metodo** (Overloading e Overriding)
2. **Polimorfismo di dati** (stessi dati in forme diverse)

### Overloading (Polimorfismo di Metodo)

Metodi con lo stesso nome ma **parametri diversi**:

```java
public class Calcolatore {
    // Overload 1
    public void stampa(String s) {
        System.out.println("String: " + s);
    }
    
    // Overload 2
    public void stampa(int i) {
        System.out.println("Int: " + i);
    }
    
    // Overload 3
    public void stampa(double d) {
        System.out.println("Double: " + d);
    }
}

Calcolatore c = new Calcolatore();
c.stampa("Ciao");    // Overload 1
c.stampa(5);         // Overload 2
c.stampa(3.14);      // Overload 3
```

### Overriding (Polimorfismo di Metodo)

Sottoclasse **riscrive** un metodo ereditato:

```java
public class Persona {
    public void descrivi() {
        System.out.println("Sono una persona");
    }
}

public class Studente extends Persona {
    @Override
    public void descrivi() {
        System.out.println("Sono uno studente");
    }
}

// Uso
Persona p = new Studente();
p.descrivi();  // Output: "Sono uno studente"
// Anche se p è dichiarato come Persona, chiama il metodo di Studente
```

### Polimorfismo di Dati

Gli stessi dati possono essere rappresentati in forme diverse:

```
Oggetto Java:
    nome: "Mario"
    eta: 25

        ↓ (Conversione ORM/Hibernate)

Riga nel Database:
    ID | Nome | Eta
    1  | Mario| 25

        ↓ (Serializzazione)

Stringa CSV:
    "Mario,25"
```

### Lista Polimorfica

Una lista che contiene oggetti di tipi diversi (ma con lo stesso supertipo):

```java
List<Persona> persone = new ArrayList<>();
persone.add(new Studente("Mario", 20));
persone.add(new Docente("Prof. Rossi", 45));
persone.add(new Persona("Luigi", 30));

// Iterazione polimorfica
for (Persona p : persone) {
    p.descrivi();  // Chiama il metodo corretto per ogni tipo
}

// Output:
// Sono uno studente
// Sono un docente
// Sono una persona
```
