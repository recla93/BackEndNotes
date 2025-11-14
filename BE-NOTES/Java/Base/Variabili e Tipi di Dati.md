Una **variabile** è un contenitore con un nome che memorizza un valore. In Java, ogni variabile deve avere un **tipo** dichiarato (Type Safety).

```java
// Dichiarazione e assegnamento insieme
int x = 5;

// Dichiarazione separata
int x;
// Assegnamento
x = 5;
```

### Tipi di Dati Primitivi

Java offre **8 tipi primitivi** fondamentali:

| Tipo | Dimensione | Intervallo | Uso |
|------|-----------|-----------|-----|
| `byte` | 1 byte | -128 a 127 | Numeri piccoli |
| `short` | 2 byte | -32.768 a 32.767 | Numeri piccoli |
| `int` | 4 byte | -2.147.483.648 a 2.147.483.647 | Numeri interi |
| `long` | 8 byte | Molto grande | Numeri molto grandi |
| `float` | 4 byte | Decimali a precisione singola | Numeri con virgola |
| `double` | 8 byte | Decimali a doppia precisione | Numeri con virgola (default) |
| `char` | 2 byte | 0 a 65535 | Caratteri unicode |
| `boolean` | 1 bit | true/false | Condizioni logiche |

### Tipi di Riferimento (Reference Types)

A differenza dei tipi primitivi, i tipi di riferimento puntano a un oggetto in memoria:

```java
String nome = "Mario";           // Riferimento a oggetto String
ArrayList<Integer> lista = new ArrayList<>();  // Riferimento a ArrayList
Persona p = new Persona();       // Riferimento a oggetto Persona
```

### Scope e Visibilità

Lo **scope** è la porzione di codice dove una variabile esiste e può essere utilizzata:

```java
public void metodo() {
    int x = 5;  // x esiste solo dentro questo metodo
    
    {
        int y = 10;  // y esiste solo dentro questo blocco
    }
    
    // System.out.println(y);  // ERRORE: y non esiste più
}
```

### Stato di una Variabile

Lo **stato** è il valore della variabile in un determinato momento del programma:

```java
int x = 5;  // Stato iniziale: x = 5
x = x + 3;  // Nuovo stato: x = 8
// Lo stato del programma è l'insieme di tutti i valori delle variabili
```