---
topic: "Collections Framework — Java"
nav_prev: "[[Array.md]]"
nav_next: "[[Boolean e Condizioni.md]]"
---

Il **Collections Framework** è l'insieme di interfacce e implementazioni per gestire gruppi di oggetti: `List` (sequenza ordinata), `Set` (senza duplicati), `Map` (chiave-valore), `Queue` (code). È parte di `java.util` ed è il modo standard di lavorare con collezioni in Java.

A differenza degli array, le collezioni hanno dimensione variabile, supportano generics e offrono algoritmi pronti (sort, search, shuffle). Tutti i framework Java (JPA, Spring, Servlet) lavorano con le collezioni.

## Gerarchia delle interfacce

```
Iterable
  └─ Collection
       ├─ List      — ArrayList, LinkedList, Vector
       ├─ Set       — HashSet, LinkedHashSet, TreeSet
       ├─ Queue     — LinkedList, PriorityQueue, ArrayDeque
       └─ Deque     — ArrayDeque, LinkedList

Map                  — HashMap, LinkedHashMap, TreeMap
  (NON estende Collection, ma fa parte del framework)
```

`Collection` è l'interfaccia base per gruppi di elementi. `Map` è separata perché lavora con coppie chiave-valore. Ogni implementazione ha caratteristiche diverse per performance, ordinamento e thread-safety.

## List — sequenza ordinata

```java
// ArrayList: array ridimensionabile (più usata)
List<String> lista = new ArrayList<>();
lista.add("a");
lista.add("b");
lista.get(0);          // "a"
lista.set(1, "c");     // sostituisce
lista.remove(0);       // per indice
lista.remove("c");     // per oggetto

// LinkedList: lista doppiamente collegata
List<String> linked = new LinkedList<>();
// insert/delete in testa/coda O(1), accesso O(n)
```

`ArrayList` è la scelta predefinita per il 90% dei casi (accesso O(1), inserimento in coda O(1) ammortizzato). `LinkedList` è utile per code frequenti in testa/coda o quando usi anche interfacce `Deque`/`Queue`.

## Set — senza duplicati

```java
// HashSet: O(1), senza ordine (usa hashCode())
Set<String> set = new HashSet<>();
set.add("a");
set.add("b");
set.add("a");          // ignorato (già presente)
set.contains("a");     // true

// LinkedHashSet: O(1), mantiene ordine di inserimento
Set<String> linked = new LinkedHashSet<>();

// TreeSet: O(log n), ordinato (Comparable o Comparator)
Set<String> sorted = new TreeSet<>();
```

`HashSet` è il più veloce ma non garantisce ordine. `LinkedHashSet` mantiene l'ordine di inserimento. `TreeSet` mantiene ordine naturale o personalizzato. Per `HashSet`, `equals()` e `hashCode()` devono essere coerenti.

## Map — chiave-valore

```java
// HashMap: O(1), senza ordine
Map<String, Integer> mappa = new HashMap<>();
mappa.put("Alice", 30);
mappa.put("Bob", 25);
mappa.get("Alice");          // 30
mappa.getOrDefault("Eve", 0); // 0
mappa.containsKey("Alice");  // true

// Iterazione
for (Map.Entry<String, Integer> entry : mappa.entrySet()) {
    entry.getKey();
    entry.getValue();
}

// LinkedHashMap: ordine di inserimento
Map<String, Integer> linked = new LinkedHashMap<>();

// TreeMap: ordinato per chiave (O(log n))
Map<String, Integer> sorted = new TreeMap<>();
```

`HashMap` è la Map più usata. `computeIfAbsent()` e `merge()` sono metodi utili per aggiornamenti atomici. Le chiavi devono essere immutabili o con `hashCode()` coerente.

## Queue e Deque

```java
// Queue: FIFO
Queue<String> coda = new LinkedList<>();
coda.offer("a");          // aggiungi
coda.poll();              // recupera e rimuovi (null se vuota)
coda.peek();              // recupera senza rimuovere

// Deque: doppia estremità
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("a");
deque.addLast("b");
deque.removeFirst();

// PriorityQueue: coda prioritaria
Queue<Integer> prioritaria = new PriorityQueue<>();
prioritaria.offer(5);
prioritaria.offer(1);
prioritaria.poll();       // 1 (il minore)
```

`ArrayDeque` è più veloce di `LinkedList` come Deque. `PriorityQueue` ordina per priorità naturale o `Comparator`. `offer()`/`poll()`/`peek()` restituiscono valori speciali su coda vuota (vs `add()`/`remove()`/`element()` che lanciano eccezioni).

## Collections utility

```java
import java.util.Collections;

List<String> list = new ArrayList<>(Arrays.asList("c", "a", "b"));

Collections.sort(list);                           // [a, b, c]
Collections.reverse(list);                        // [c, b, a]
Collections.shuffle(list);                        // ordine casuale
Collections.binarySearch(list, "b");              // ricerca binaria
Collections.frequency(list, "a");                 // conteggio occorrenze

// Collezioni immodificabili
List<String> readOnly = Collections.unmodifiableList(list);
// list.add(...) → UnsupportedOperationException

// Collezioni sincronizzate (per thread-safety)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```

`Collections` è la classe utility per algoritmi su collezioni. Per collezioni immutabili, preferisci `List.of()`, `Set.of()`, `Map.of()` (Java 9+).

## Factory methods (Java 9+)

```java
// Collezioni immutabili (più compatte)
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Map<String, Integer> mapEntries = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);
```

`List.of()`, `Set.of()`, `Map.of()` creano collezioni immutabili (non modificabili, non accettano null). Più compatte e performanti di `Arrays.asList()`.

## Tabella riassuntiva

| Interfaccia | Implementazione | Ordine | Performance | Thread-safe |
|-------------|----------------|--------|-------------|-------------|
| `List` | `ArrayList` | Inserimento | O(1) accesso | No |
| `List` | `LinkedList` | Inserimento | O(n) accesso | No |
| `Set` | `HashSet` | Nessuno | O(1) | No |
| `Set` | `TreeSet` | Naturale | O(log n) | No |
| `Map` | `HashMap` | Nessuno | O(1) | No |
| `Map` | `TreeMap` | Per chiave | O(log n) | No |
| `Queue` | `PriorityQueue` | Priorità | O(log n) | No |
| — | `ConcurrentHashMap` | Nessuno | O(1) | **Si** |
| — | `CopyOnWriteArrayList` | Inserimento | O(n) | **Si** |

## Errori comuni

- **Modificare una collezione mentre la iteri**: `ConcurrentModificationException`. Usa `Iterator.remove()` o `removeIf()`.
- **Usare `==` per confronto oggetti in Set/Map**: usa `equals()`/`hashCode()` coerenti.
- **Dimenticare generics**: `List list = new ArrayList()` causa cast espliciti e warning.
- **`Set`/`Map` senza `hashCode()` corretto**: elementi duplicati o non trovati. Se override `equals()`, override sempre `hashCode()`.
- **ArrayList con add/remove in testa**: `list.add(0, x)` è O(n). Usa `LinkedList` o `ArrayDeque`.
- **Collections.unmodifiable... è superficiale**: non rende immutabili gli oggetti contenuti.

## Best Practices & Conventions

- **Programma alle interfacce**: `List<String> list = new ArrayList<>()`, non `ArrayList<String> list`.
- **Scegli l'implementazione giusta**: `ArrayList` per accesso casuale, `LinkedList` per code frequenti.
- **Usa `List.of()` per costanti** (Java 9+).
- **Per campi immutabili**: `Collections.unmodifiableList()` o `List.copyOf()`.
- **Inizializza con capacità iniziale** se conosci la dimensione approssimativa: `new ArrayList<>(100)`.
- **Preferisci `Deque` a `Stack`**: `Stack` (legacy) è sincronizzato e lento. `ArrayDeque` è più veloce.
- **Usa `computeIfAbsent()` per mappe con valori complessi**: evita il pattern `if (!map.containsKey(k)) map.put(k, new Value())`.
