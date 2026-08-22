---
topic: "Array — Java"
nav_prev: "[[Stringhe.md]]"
nav_next: "[[Collections Framework.md]]"
---

Gli array in Java sono contenitori a **dimensione fissa** di elementi omogenei. Sono oggetti (ereditano da `Object`), ma con sintassi speciale. A differenza delle `List`, la dimensione non può cambiare dopo la creazione.

Esistono array di primitivi (`int[]`, `char[]`) e array di oggetti (`String[]`, `Integer[]`). `int[]` memorizza valori direttamente; `Integer[]` memorizza riferimenti a oggetti.

## Dichiarazione e inizializzazione

```java
// Dichiarazione (tipi)
int[] numeri;          // preferita
int numeri2[];         // valida (stile C, sconsigliata)

// Inizializzazione con dimensione fissa
numeri = new int[5];   // [0, 0, 0, 0, 0] (inizializzato a default)

// Inizializzazione con valori
int[] valori = {10, 20, 30, 40, 50};

// Array di oggetti
String[] nomi = new String[3];  // [null, null, null]
String[] frutti = {"mela", "pera", "banana"};
```

Gli array di primitivi sono inizializzati a `0`/`0.0`/`false`. Gli array di oggetti sono inizializzati a `null`. La sintassi `{...}` funziona solo nella dichiarazione, non in riassegnazione.

## Accesso e iterazione

```java
int[] numeri = {10, 20, 30, 40, 50};

numeri[0]               // 10
numeri[numeri.length - 1]  // 50 (ultimo)
numeri[5]               // ArrayIndexOutOfBoundsException!

// Iterazione classica
for (int i = 0; i < numeri.length; i++) {
    System.out.println(numeri[i]);
}

// For-each (più leggibile)
for (int n : numeri) {
    System.out.println(n);
}
```

`length` è un campo (non un metodo come in `String`). L'indice parte da `0`. L'accesso con indice negativo o >= `length` lancia `ArrayIndexOutOfBoundsException` a runtime.

## Array multidimensionali

```java
// Matrice 3x4
int[][] matrice = new int[3][4];
matrice[1][2] = 5;

// Inizializzazione
int[][] identita = {
    {1, 0, 0},
    {0, 1, 0},
    {0, 0, 1}
};

// Array irregolari (jagged array)
int[][] triangolo = new int[4][];
triangolo[0] = new int[1];
triangolo[1] = new int[2];
triangolo[2] = new int[3];
triangolo[3] = new int[4];
```

In Java gli array multidimensionali sono **array di array** (non un blocco contiguo come in C). Ogni riga può avere lunghezza diversa (jagged array).

## java.util.Arrays

```java
import java.util.Arrays;

int[] numeri = {42, 7, 13, 7, 99};

Arrays.sort(numeri);                     // [7, 7, 13, 42, 99]
Arrays.binarySearch(numeri, 13);         // 2 (array deve essere ordinato)
Arrays.toString(numeri);                 // "[7, 7, 13, 42, 99]"
Arrays.equals(numeri, altri);            // confronto contenuto
Arrays.fill(numeri, 0);                  // tutti a 0
Arrays.copyOf(numeri, 10);               // copia con nuova lunghezza
Arrays.stream(numeri).filter(n -> n > 10).toArray();  // Java 8+
```

La classe `Arrays` è il toolbelt per array: sort, ricerca, copia, confronto, fill, stream. `Arrays.equals()` confronta il contenuto (a differenza di `array1.equals(array2)` che confronta riferimenti).

## Copia di array

```java
int[] original = {1, 2, 3, 4, 5};

// Shallow copy
int[] copia1 = original.clone();                   // OK per primitivi
int[] copia2 = Arrays.copyOf(original, original.length);
int[] copia3 = new int[5];
System.arraycopy(original, 0, copia3, 0, 5);       // low-level

// Per array di oggetti: shallow copy (gli oggetti NON sono clonati)
String[] parole = {"a", "b"};
String[] copiaParole = parole.clone();  // stesso array di oggetti
```

`clone()` e `copyOf()` fanno shallow copy. Per array di primitivi è una copia completa. Per array di oggetti, i riferimenti vengono copiati, non gli oggetti stessi.

## Array vs List

```java
// Array: dimensione fissa, performance migliore
int[] numeri = new int[1000];
numeri[0] = 42;

// List: dimensione variabile, più flessibile
List<Integer> lista = new ArrayList<>(1000);
lista.add(42);

// Conversione
String[] array = {"a", "b", "c"};
List<String> list = Arrays.asList(array);          // vista: modifiche si riflettono
List<String> modifiable = new ArrayList<>(list);    // copia indipendente
String[] back = list.toArray(new String[0]);        // List → Array
```

Preferisci `List` per API pubbliche (flessibilità). Usa array per: performance critiche, dati primitivi in grandi quantità, intercambio con API legacy.

## Errori comuni

- **`ArrayIndexOutOfBoundsException`**: indice fuori range. Controlla sempre `length - 1` per l'ultimo elemento.
- **Confondere `length` (campo) con `length()` (metodo)**: array usano `.length`, String usa `.length()`.
- **Array di oggetti non inizializzati**: `String[] s = new String[5]; s[0].length()` → `NullPointerException`.
- **Pensare che `array1.equals(array2)` confronti il contenuto**: confronta riferimenti. Usa `Arrays.equals()`.
- **Modificare l'array attraverso `Arrays.asList()`**: la vista è fixed-size, `add()`/`remove()` lanciano `UnsupportedOperationException`.
- **Confondere array e varargs**: `void metodo(String... args)` riceve un array. Puoi passare array o lista separata da virgole.

## Best Practices & Conventions

- Per collezioni di dimensione variabile, preferisci `List`/`Set`/`Map`.
- Usa array per: dati primitivi in grandi volumi, interoperabilità con API native/legacy.
- Usa `Arrays.toString()` e `Arrays.deepToString()` per logging/debug.
- Per array di oggetti, considera che `equals()` e `hashCode()` usano l'identità, non il contenuto.
- Non esporre array pubblicamente: restituisci una copia (`return array.clone()`) o una `List` immodificabile.
- Per ordinamento, `Arrays.sort()` usa Dual-Pivot Quicksort per primitivi e TimSort per oggetti.
