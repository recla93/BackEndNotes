---
topic: "Generics"
nav_prev: "[[Costrutti di Iterazione.md]]"
nav_next: "[[Method Class.md]]"
---

I **generics** (introdotti in Java 5) permettono di parametrizzare i tipi: scrivi codice che funziona con **qualsiasi tipo**, ma con type-safety in fase di compilazione. Prima dei generics si usava `Object` con cast espliciti — pericolosi e verbosi.

## Prima e dopo

```java
// ❌ SENZA generics: cast manuale, errore solo a runtime
List lista = new ArrayList();
lista.add("Ciao");
String s = (String) lista.get(0);  // cast obbligatorio
lista.add(123);                     // nessun errore in compilazione
String s2 = (String) lista.get(1); // ClassCastException a RUNTIME!

// ✅ CON generics: type-safe in compilazione
List<String> lista = new ArrayList<>();
lista.add("Ciao");
String s = lista.get(0);            // nessun cast
lista.add(123);                      // ERRORE in compilazione!
```

## Quando usarli

- **Sempre** quando lavori con collezioni (`List<T>`, `Map<K,V>`, `Set<E>`)
- **Classi utility** — `Optional<T>`, `Comparable<T>`, `Comparator<T>`
- **Repository/Services** — `JpaRepository<T, ID>`, `CrudRepository<T, ID>`
- **DTO generici** — `ApiResponse<T>`, `Page<T>`
- **Metodi di mapping** — `EntityMapper.toDto(T entity)`

## Notazione standard

| Nome | Significato | Esempio |
|---|---|---|
| `T` | Type (tipo generico) | `List<T>`, `Optional<T>` |
| `E` | Element (collezioni) | `List<E>`, `Set<E>` |
| `K`, `V` | Key, Value (mappe) | `Map<K, V>` |
| `R` | Return (tipo di ritorno) | `Supplier<R>` |
| `N` | Number | `Box<N extends Number>` |

## Classi e interfacce generiche

```java
public class Box<T> {
    private T content;

    public void set(T content) { this.content = content; }
    public T get() { return content; }
}

// Uso
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String val = stringBox.get();  // nessun cast

Box<Integer> intBox = new Box<>();
intBox.set(42);
```

## Metodi generici

```java
public <T> T getFirst(List<T> list) {
    return list.get(0);
}

// Uso — inferenza del tipo
String s = getFirst(List.of("a", "b"));  // T è String
Integer i = getFirst(List.of(1, 2, 3));  // T è Integer
```

## Bounded Type Parameters (vincoli)

Limiti il tipo a una specifica gerarchia:

```java
// Upper bound: T deve essere Number o un suo sottotipo (Integer, Double, etc.)
public <T extends Number> double sum(List<T> numbers) {
    double total = 0;
    for (T n : numbers) total += n.doubleValue();
    return total;
}

// Multipli bound: T deve estendere Entity E implementare Comparable
public <T extends BaseEntity & Comparable<T>> void sort(List<T> entities) { ... }
```

## Wildcards (?), PECS e covariance

```java
// ? extends T — produci (leggi) oggetti di tipo T o sottotipi
List<? extends Number> numbers = List.of(1, 2.5, 3L);
Number n = numbers.get(0);   // OK: leggi come Number
// numbers.add(42);          // ERRORE: non puoi aggiungere

// ? super T — consumi (scrivi) oggetti di tipo T o supertipi
List<? super Number> sink = new ArrayList<>();
sink.add(42);                // OK: scrivi Number (o Integer)
sink.add(3.14);
// Number n = sink.get(0);  // ERRORE: non sai che tipo è

// ? unbounded — qualsiasi tipo (read-only)
public void print(List<?> list) {
    for (Object o : list) System.out.println(o);
}
```

**PECS** — Producer Extends, Consumer Super:
- Se il metodo **legge** dati dalla struttura → `? extends T`
- Se il metodo **scrive** dati nella struttura → `? super T`
- Se fa entrambi → nessun wildcard, usa `T`

## Problemi comuni

1. **Type erasure** — i generics esistono solo in compilazione. A runtime, `List<String>` e `List<Integer>` sono la stessa cosa (`List`). Non puoi fare `instanceof` di un tipo generico.
2. **Non si possono creare array di tipo generico**: `new T[size]` non compila. Usa `List<T>` invece.
3. **Non si possono usare primitivi**: `List<int>` non compila. Usa `List<Integer>` (autoboxing).
4. **Overloading ambiguo**: `void process(List<String>)` e `void process(List<Integer>)` non compilano insieme (type erasure → stessa firma).
5. **Static context**: i campi statici non possono usare il parametro `T` della classe.

```java
public class Box<T> {
    // ❌ Non compila: type erasure cancella T a runtime
    // public static T getDefault() { ... }

    // ✅ Alternativa: passa il tipo esplicitamente
    public static <T> T getDefault(Class<T> clazz) throws Exception {
        return clazz.getDeclaredConstructor().newInstance();
    }
}
```