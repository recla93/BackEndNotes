---
topic: "Comparable e Comparator — Java"
nav_prev: "[[Generics.md]]"
nav_next: "[[Costrutti di Iterazione.md]]"
---

`Comparable` e `Comparator` sono le due interfacce per ordinare oggetti in Java. `Comparable` definisce l'**ordine naturale** di una classe (confronto `this` vs `other`). `Comparator` definisce un **ordine esterno** separato, utile per criteri multipli o classi di terze parti.

`Comparable` si implementa sulla classe stessa (modificandola). `Comparator` è una classe separata o lambda. Entrambi usano `Collections.sort()` e `TreeSet`/`TreeMap`.

## Comparable — ordine naturale

```java
public class Persona implements Comparable<Persona> {
    private String nome;
    private int eta;

    @Override
    public int compareTo(Persona altra) {
        return Integer.compare(this.eta, altra.eta);
    }
}

// Uso
List<Persona> persone = new ArrayList<>();
Collections.sort(persone);  // ordina per eta
```

`compareTo()` restituisce: negativo se `this < other`, zero se uguali, positivo se `this > other`. `Integer.compare()` e `String.compareTo()` sono helper safe per evitare overflow.

## Comparator — ordine esterno

```java
// Con classe anonima (pre-Java 8)
Comparator<Persona> perNome = new Comparator<>() {
    @Override
    public int compare(Persona a, Persona b) {
        return a.getNome().compareTo(b.getNome());
    }
};

// Con lambda (Java 8+)
Comparator<Persona> perNome = (a, b) -> a.getNome().compareTo(b.getNome());

// Con Comparator.comparing() (Java 8+)
Comparator<Persona> perNome = Comparator.comparing(Persona::getNome);
Comparator<Persona> perEta = Comparator.comparingInt(Persona::getEta);

// Uso
Collections.sort(persone, perNome);
persone.sort(perNome);  // List.sort() (Java 8+)
```

`Comparator.comparing()` accetta method reference e genera automaticamente il `Comparator`. `comparingInt()`, `comparingLong()`, `comparingDouble()` evitano boxing. `List.sort()` (Java 8+) è più compatto di `Collections.sort()`.

## Comparator chain (criteri multipli)

```java
List<Persona> persone = List.of(
    new Persona("Alice", 30),
    new Persona("Bob", 25),
    new Persona("Alice", 25)
);

// Ordina per nome, poi per età
Comparator<Persona> perNomePoiEta = Comparator
    .comparing(Persona::getNome)
    .thenComparingInt(Persona::getEta);

persone.sort(perNomePoiEta);
// Alice(25), Alice(30), Bob(25)

// Ordine inverso
Comparator<Persona> perEtaDecrescente = Comparator
    .comparingInt(Persona::getEta)
    .reversed();
```

`thenComparing()` incatena criteri secondari. `reversed()` inverte l'ordine. Puoi combinare `nullsFirst()`/`nullsLast()` per gestire valori null.

## Null handling

```java
Comparator<Persona> perNomeConNull = Comparator
    .nullsFirst(Comparator.comparing(Persona::getNome));

// nullsFirst: i null vanno prima (o dopo con nullsLast)
List<Persona> conNull = new ArrayList<>();
conNull.add(null);
conNull.add(new Persona("Bob", 25));
conNull.sort(perNomeConNull);  // [null, Bob(25)]
```

`nullsFirst()` e `nullsLast()` sono wrapper che gestiscono elementi null nella collezione. Senza, `compare()` lancia `NullPointerException`.

## Ordine naturale con campi multipli

```java
public class Prodotto implements Comparable<Prodotto> {
    private String categoria;
    private double prezzo;

    @Override
    public int compareTo(Prodotto altro) {
        // Per categoria, poi per prezzo
        int catCmp = this.categoria.compareTo(altro.categoria);
        if (catCmp != 0) return catCmp;
        return Double.compare(this.prezzo, altro.prezzo);
    }
}
```

Implementa `Comparable` quando c'è un ordine naturale ovvio (date, id, nome logico). Per criteri multipli, la chain con `Comparator` è spesso più pulita che implementare `Comparable` complesso.

## Ordinamento in stream (Java 8+)

```java
List<Persona> ordinata = persone.stream()
    .filter(p -> p.getEta() > 18)
    .sorted(Comparator.comparing(Persona::getNome))
    .collect(Collectors.toList());

// Min/Max con Comparator
Persona piuGiovane = persone.stream()
    .min(Comparator.comparingInt(Persona::getEta))
    .orElseThrow();
```

`Stream.sorted()` accetta un `Comparator` opzionale. `Stream.min()`/`max()` con `Comparator` trovano l'elemento minimo/massimo.

## Tabella comparativa

| Caratteristica | Comparable | Comparator |
|---------------|------------|------------|
| Package | java.lang | java.util |
| Metodo | `compareTo(T)` | `compare(T, T)` |
| Modifica la classe | Si | No |
| Criteri multipli | Complesso | Facile (chain) |
| Lambda/stream | No | Si |
| Null handling | Manuale | `nullsFirst()`/`nullsLast()` |
| Ordinamento multiplo | Una sola | Quanti vuoi |
| Uso tipico | Ordine naturale | Ordinamenti diversi |

## Errori comuni

- **Non coerente con `equals()`**: se `a.compareTo(b) == 0` ma `!a.equals(b)`, i Set/Map ordinati (TreeSet, TreeMap) potrebbero comportarsi inaspettatamente.
- **Dimenticare il tipo generico**: `implements Comparable` senza `<Persona>` usa raw type e richiede cast.
- **Overflow in sottrazione**: `return a.eta - b.eta` può overflow per valori estremi. Usa `Integer.compare()`.
- **`Comparator` che lancia `NullPointerException`**: usa `Comparator.nullsFirst()` o controlla esplicitamente.
- **Modificare l'ordinamento di un elemento già in un TreeSet**: `TreeSet` non si riordina automaticamente. Rimuovi e reinserisci.

## Best Practices & Conventions

- Usa `Comparable` per ordine naturale (date, id, codici univoci).
- Usa `Comparator` per: classi di terze parti, criteri multipli, ordinamenti temporanei.
- Preferisci `Comparator.comparing()` e lambda: più leggibili e compatti.
- Usa `Comparator.nullsFirst()`/`nullsLast()` per gestire elementi null.
- Assicurati che `compareTo()` sia coerente con `equals()` per l'uso in `TreeSet`/`TreeMap`.
- Per ordinamenti composti, usa `.thenComparing()` invece di implementare logica manuale.
