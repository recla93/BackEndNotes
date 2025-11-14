
Un'**espressione** è qualsiasi costrutto che **produce un valore**:

```java
int x = 5 + 3;        // 5 + 3 è un'espressione che produce 8
String s = "Ciao" + " " + "Mario";  // Concatenazione, produce "Ciao Mario"
boolean b = 10 > 5;   // Espressione booleana, produce true
```

### Operazioni Aritmetiche

```java
// Con tipo int (arrotondamento per difetto)
int x = 5 / 2;        // Risultato: 2 (NON 2.5!)
int y = 10 % 3;       // Modulo: 1

// Con tipo double (decimali)
double a = 5.0 / 2.0;  // Risultato: 2.5
double b = 10.5 % 3;   // Modulo con decimali
```

### Concatenazione di String

Java esegue le operazioni **da sinistra verso destra**:

```java
// Esempio 1: numero + numero + stringa
System.out.println(10 + 5 + "ciao");  // Output: 15ciao
// (10+5 = 15, poi concatena con "ciao")

// Esempio 2: stringa + numero + numero
System.out.println("ciao" + 10 + 5);  // Output: ciao105
// (tutto diventa stringa, concatenazione)

// Esempio 3: uso delle parentesi
System.out.println("ciao" + (10 + 5));  // Output: ciao15
// (parentesi hanno priorità)
```

### Operatori di Assegnamento Composto

```java
x += 5;    // Equivalente a: x = x + 5
x -= 3;    // Equivalente a: x = x - 3
x *= 2;    // Equivalente a: x = x * 2
x /= 4;    // Equivalente a: x = x / 4
```