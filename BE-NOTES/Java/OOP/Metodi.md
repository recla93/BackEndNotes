### Firma del Metodo

```
[visibilità] [static] tipoRitorno nomeMetodo ([parametri]) {
    // corpo del metodo
}
```

Esempio:

```java
public static int somma(int a, int b) {
    return a + b;
}
```

### Parametri e Return

```java
// Con ritorno
public int somma(int a, int b) {
    return a + b;  // Ritorna il risultato
}

// Senza ritorno (void)
public void stampa() {
    System.out.println("Ciao");  // Non ritorna nulla
}

// Uso
int risultato = somma(5, 3);  // risultato = 8
stampa();  // Esecuzione senza cattura del valore
```

### Metodi Static

I metodi **static** appartengono alla **classe**, non all'oggetto. Esistono una sola volta in memoria:

```java
public class Matematica {
    // Metodo di classe (static)
    public static int somma(int a, int b) {
        return a + b;
    }
    
    // Metodo di oggetto (non static)
    public int moltiplicazione(int a, int b) {
        return a * b;
    }
}

// Uso
int s1 = Matematica.somma(5, 3);  // Metodo static sulla classe
Matematica m = new Matematica();
int s2 = m.moltiplicazione(5, 3);  // Metodo di oggetto sull'istanza
```

### Overload di Metodi

Metodi con lo stesso nome ma **parametri diversi**:

```java
public class Calcolatore {
    // Overload 1: due interi
    public int somma(int a, int b) {
        return a + b;
    }
    
    // Overload 2: tre interi
    public int somma(int a, int b, int c) {
        return a + b + c;
    }
    
    // Overload 3: due double
    public double somma(double a, double b) {
        return a + b;
    }
}

// Uso
Calcolatore c = new Calcolatore();
c.somma(5, 3);           // Usa overload 1
c.somma(5, 3, 2);        // Usa overload 2
c.somma(5.5, 3.2);       // Usa overload 3
```
