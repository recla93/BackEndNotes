### Cos'è un Enum?

Un **enum** è un insieme di **costanti predefinite**. Ogni valore dell'enum è un'istanza unica della classe enum:

```java
public enum Colore {
    ROSSO,
    VERDE,
    BLU
}

// Uso
Colore c = Colore.ROSSO;  // Unica istanza di ROSSO (Singleton)
```

### Enum con Proprietà e Metodi

```java
public enum Giorno {
    LUNEDI(1, "Lunedì"),
    MARTEDI(2, "Martedì"),
    MERCOLEDI(3, "Mercoledì"),
    GIOVEDI(4, "Giovedì"),
    VENERDI(5, "Venerdì"),
    SABATO(6, "Sabato"),
    DOMENICA(7, "Domenica");
    
    private int numero;
    private String nome;
    
    // Costruttore (privato per default)
    Giorno(int numero, String nome) {
        this.numero = numero;
        this.nome = nome;
    }
    
    // Getter
    public int getNumero() { return numero; }
    public String getNome() { return nome; }
}

// Uso
Giorno g = Giorno.LUNEDI;
System.out.println(g.getNome());    // "Lunedì"
System.out.println(g.getNumero());  // 1
```

### Switch con Enum

```java
public void impostaGiorno(Giorno g) {
    switch(g) {
        case LUNEDI, MARTEDI, MERCOLEDI, GIOVEDI, VENERDI ->
            System.out.println("Giorni lavorativi");
        case SABATO, DOMENICA ->
            System.out.println("Weekend");
    }
}
```

### Vantaggi degli Enum

- ✓ **Sicurezza di tipo**: il compilatore verifica i valori
- ✓ **Completezza**: il compilatore avvisa se manca un caso
- ✓ **Readibilità**: il codice è più chiaro
- ✓ **Singleton**: ogni valore è un'istanza unica
- ✓ **Iterate**: puoi scorrere tutti i valori

```java
for (Giorno g : Giorno.values()) {
    System.out.println(g.getNome());
}
```
