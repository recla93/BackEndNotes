# Lambda, Stream e Classi Anonime

Java 8 ha introdotto la programmazione funzionale con lambda e stream, riducendo drasticamente la verbosità del codice.

## Classe Anonima

Una classe **anonima** è una classe senza nome, dichiarata e istanziata in una singola espressione. Utile quando serve un'implementazione usa-e-getta.

```java
// Prima di Java 8: classe anonima per definire un comportamento
Comparable<String> comp = new Comparable<String>() {
    @Override
    public int compareTo(String o) {
        return this.length() - o.length();
    }
};

// Classe anonima per un thread
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Ciao da thread anonimo");
    }
};
new Thread(task).start();
```

**Quando usare:** implementazioni monouso di interfacce con 1-2 metodi (callback, listener, comparator).

**Problema:** molto verboso — serve scrivere `new NomeInterfaccia() { @Override ... }` anche per una singola riga.

## Lambda Expressions (Java 8+)

Una **lambda** è una versione compatta di una classe anonima per interfacce **funzionali** (con un solo metodo astratto).

```java
// Lambda: parametro -> espressione
// (parametri) -> { corpo }

// Esempi:
Runnable task = () -> System.out.println("Ciao");
Comparator<String> comp = (s1, s2) -> s1.length() - s2.length();

// Con blocchi
BiFunction<Integer, Integer, Integer> max = (a, b) -> {
    if (a > b) return a;
    return b;
};
```

**Equivalenza:**
```java
// Classe anonima
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Ciao");
    }
};

// Lambda
Runnable r2 = () -> System.out.println("Ciao");

// Method reference (ancora più corto se richiami un metodo esistente)
Runnable r3 = System.out::println;
```

**Quando usare:** sempre che devi implementare un'interfaccia funzionale (Predicate, Consumer, Function, Supplier, Runnable, Comparator).

## Interfacce Funzionali Principali

| Interfaccia | Metodo | Input | Output | Uso |
|---|---|---|---|---|
| `Predicate<T>` | `test(T)` | T | boolean | Filtri |
| `Consumer<T>` | `accept(T)` | T | void | Operazioni senza ritorno |
| `Function<T,R>` | `apply(T)` | T | R | Trasformazioni |
| `Supplier<T>` | `get()` | Nessuno | T | Produci valori |
| `Comparator<T>` | `compare(T,T)` | T,T | int | Ordinamento |
| `Runnable` | `run()` | Nessuno | void | Thread |
| `Callable<T>` | `call()` | Nessuno | T | Thread con risultato |

## Stream API

Stream è una sequenza di elementi che supporta operazioni aggregate. Non è una collezione — i dati non vengono memorizzati, ma **processati**.

```java
List<String> nomi = List.of("Mario", "Anna", "Giovanni", "Luca", "Beatrice");

// Catena di operazioni
List<String> filtrati = nomi.stream()               // crea stream
    .filter(n -> n.length() > 4)                    // intermedio: filtra
    .map(String::toUpperCase)                       // intermedio: trasforma
    .sorted()                                       // intermedio: ordina
    .collect(Collectors.toList());                  // terminale: raccogli

// ["BEATRICE", "GIOVANNI", "MARIO"]
```

**Operazioni intermedie** (restituiscono uno stream, si possono concatenare): `filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `skip`, `peek`

**Operazioni terminali** (producono un risultato o effetto collaterale): `collect`, `forEach`, `count`, `reduce`, `anyMatch`, `allMatch`, `noneMatch`, `findFirst`, `findAny`

### Filter — selezionare elementi

```java
List<String> nomiLunghi = nomi.stream()
    .filter(n -> n.length() > 4)
    .collect(Collectors.toList());
```

### Map — trasformare elementi

```java
List<Integer> lunghezze = nomi.stream()
    .map(String::length)
    .collect(Collectors.toList());
```

### Optional — evitare null pointer

`Optional<T>` è un contenitore che può contenere o meno un valore. Obbliga a gestire il caso vuoto.

```java
Optional<String> optional = nomi.stream()
    .filter(n -> n.startsWith("Z"))
    .findFirst();

// Gestione
String risultato = optional.orElse("Nessun nome trovato");
// optional.orElseThrow(() -> new NotFoundException("..."));
// optional.ifPresent(System.out::println);
```

### Operazioni di riduzione

```java
// Contare
long count = nomi.stream().filter(n -> n.length() > 3).count();

// Somma
int totale = List.of(1, 2, 3).stream().reduce(0, Integer::sum);  // 6

// Raccogliere in mappa
Map<Integer, List<String>> raggruppati = nomi.stream()
    .collect(Collectors.groupingBy(String::length));
```

## Quando usare cosa

| Situazione | Soluzione |
|---|---|
| Filtrare una lista | `stream().filter().collect()` |
| Trasformare elementi | `stream().map().collect()` |
| Iterare senza modificare | `forEach()` o for-each |
| Ordinare | `stream().sorted()` |
| Trovare un elemento | `stream().filter().findFirst()` |
| Verificare condizione | `anyMatch()`, `allMatch()`, `noneMatch()` |
| Raggruppare | `Collectors.groupingBy()` |
| Unire due liste | `Stream.concat(a, b).collect()` |

## Performance

| Operazione | For classico | Stream |
|---|---|---|
| Filter + collect | 1x (baseline) | ~1.2-2x più lento |
| Filter + map + reduce | 1x | ~1.5x più lento (ma più leggibile) |
| Parallel stream (multi-core) | N/A | Più veloce su dataset grandi |
| Singolo elemento | 1x | ~2-3x overhead |
| Operazioni semplici (es. somma) | For con primitivi | Meglio for |

**Regola:** stream per leggibilità su operazioni complesse e dataset medi/grandi. For classico per loop semplici o hot path critici.

## Problemi comuni

| Problema | Esempio | Soluzione |
|---|---|---|
| **Modifica variabili esterne** | `int x = 0; list.forEach(e -> x++)` | Le variabili devono essere effectively final |
| **Side effects in stream** | `stream().peek(System.out::println).collect(...)` | Usa `forEach` per side effects |
| **Null in stream** | `stream.filter(s -> s.length() > 3)` con null | Filtra null prima: `.filter(Objects::nonNull)` |
| **Parallel stream non thread-safe** | `stream.parallel().collect(toList())` | Usa `ConcurrentHashMap` o collezioni thread-safe |
| **Troppi stream annidati** | Leggibilità peggiora | Estrai variabili intermedie |