---
topic: "Object class — metodi fondamentali"
nav_prev: "[[Comparable Comparator.md]]"
nav_next: "[[Boolean e Condizioni.md]]"
---

`Object` è la superclasse di tutte le classi Java. I suoi metodi sono ereditati da ogni oggetto. I più importanti da conoscere e spesso da override: `equals()`, `hashCode()`, `toString()`, `clone()`, `finalize()`.

`equals()` e `hashCode()` hanno un contratto preciso: oggetti uguali secondo `equals()` devono avere lo stesso `hashCode()`. Violare questo contratto rompe `HashMap`, `HashSet`, `HashTable`.

## equals() — uguaglianza logica

```java
public class Persona {
    private String nome;
    private int eta;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Persona persona = (Persona) o;
        return eta == persona.eta && Objects.equals(nome, persona.nome);
    }

    @Override
    public int hashCode() {
        return Objects.hash(nome, eta);
    }
}
```

Il contratto di `equals()`: riflessivo, simmetrico, transitivo, consistente, e `x.equals(null)` deve essere `false`. `Objects.equals(a, b)` gestisce i null (evita `NullPointerException`).

## hashCode() — impronta per HashMap

```java
@Override
public int hashCode() {
    return Objects.hash(nome, eta);  // Java 7+, gestisce null
}

// Manuale (più performante)
@Override
public int hashCode() {
    int result = nome != null ? nome.hashCode() : 0;
    result = 31 * result + eta;
    return result;
}
```

`Objects.hash()` è compatto ma crea un array varargs ad ogni chiamata. La versione manuale è più performante per chiamate frequenti. Il numero primo 31 è una convenzione Java (buona distribuzione, calcolabile come `(i << 5) - i`).

## toString() — rappresentazione leggibile

```java
@Override
public String toString() {
    return "Persona{" +
           "nome='" + nome + '\'' +
           ", eta=" + eta +
           '}';
}

// Con Objects.toString() (safe per null)
@Override
public String toString() {
    return String.format("Persona{nome='%s', eta=%d}",
        Objects.toString(nome, "?"), eta);
}
```

`toString()` è chiamato implicitamente in `System.out.println(obj)`, log e debug. Se non override, stampa `ClassName@hashCode` (in esadecimale). Usa `String.format()` o `Objects.toString()` per gestire campi null.

## Lombok: @Data, @EqualsAndHashCode, @ToString

```java
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data                           // @Getter @Setter @ToString @EqualsAndHashCode @RequiredArgsConstructor
public class Persona {
    private String nome;
    private int eta;
}

// Escludere campi problematici
@Data
@EqualsAndHashCode.Exclude campo = "eta"
public class Utente {
    private Long id;
    private String nome;
}
```

Lombok genera automaticamente `equals()`, `hashCode()`, `toString()`. Attenzione su entità JPA con relazioni lazy: `@Data` genera `toString()` che carica tutte le relazioni (N+1). Preferisci `@Getter`/`@Setter` espliciti per entità.

## clone() — copia dell'oggetto

```java
public class Persona implements Cloneable {
    private String nome;
    private Indirizzo indirizzo;  // oggetto mutabile

    @Override
    public Persona clone() throws CloneNotSupportedException {
        Persona cloned = (Persona) super.clone();         // shallow copy
        cloned.indirizzo = this.indirizzo.clone();         // deep copy per campi mutabili
        return cloned;
    }
}
```

`clone()` è controverso: `Cloneable` è un marker interface senza metodi, `clone()` è `protected` su `Object`. La shallow copy copia i riferimenti (stessi oggetti interni). Per deep copy considera: costruttore di copia, `copyOf()`, o una libreria come `Apache Commons Lang3`.

## equals() e hashCode() su entità JPA

```java
@Entity
public class Utente {
    @Id
    private Long id;
    private String nome;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Utente utente = (Utente) o;
        return id != null && Objects.equals(id, utente.id);
    }

    @Override
    public int hashCode() {
        // Usa solo l'ID (o una costante per non ancora persistenti)
        return getClass().hashCode();
    }
}
```

Per entità JPA, usa solo l'ID (o una costante per entità non ancora salvate). Non includere campi business in `equals()`/`hashCode()`: cambiano nel tempo e rompono la persistenza in Set/Map dopo merge.

## Errori comuni

- **`equals()` senza `hashCode()`**: viola il contratto. HashMap/Set non funzionano correttamente.
- **Usare campi mutabili in `hashCode()`**: se un campo cambia dopo essere stato inserito in una HashMap, l'oggetto è "perso" (non trovabile).
- **`toString()` che causa effetti collaterali**: su entità JPA, `toString()` carica relazioni lazy. Stack overflow se ci sono relazioni circolari.
- **Dimenticare `@Override`**: se il metodo ha firma diversa, non stai facendo override ma overload.
- **`clone()` superficiale per oggetti mutabili**: la copia condivide gli oggetti interni. Usa deep copy.
- **`finalize()` non prevedibile**: deprecato da Java 9. Non usare mai. Usa `Cleaner` o `AutoCloseable`.

## Best Practices & Conventions

- Override sempre **entrambi** `equals()` e `hashCode()` insieme, mai uno senza l'altro.
- Usa **`Objects.equals()`** e **`Objects.hash()`** per implementazioni safe contro null.
- Per entità JPA, usa `equals()` basato solo sull'ID.
- Per DTO e value object, usa **Lombok `@Data`** o **Java Records** (che generano automaticamente).
- Non usare `clone()` per codice nuovo. Preferisci costruttori di copia o `copyOf()`.
- Override `toString()` per debugging e logging: stampa campi significativi in formato chiaro.
