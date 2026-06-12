# Override, Overload, Interfaces

Tre concetti fondamentali dell'OOP in Java: **override** (sostituire), **overload** (sovraccaricare), **interfacce** (contratti).

## Override — ridefinire un metodo ereditato

Un metodo **override** ha la stessa firma (nome + parametri) di un metodo della superclasse, ma implementazione diversa.

```java
public class Animale {
    public void verso() {
        System.out.println("Verso generico");
    }
}

public class Cane extends Animale {
    @Override
    public void verso() {               // stessa firma
        System.out.println("Bau!");
    }
}

// Uso
Animale a = new Cane();
a.verso();  // "Bau!" — a runtime usa il metodo del Cane (dynamic dispatch)
```

**Regole dell'override:**
- Firma identica (nome + parametri)
- Visibilità non può essere più restrittiva (`public` → `public`, non `public` → `private`)
- Il metodo sovrascritto può lanciare eccezioni più specifiche o nessuna
- Usa sempre `@Override` — il compilatore controlla che la firma sia corretta

**Problemi comuni:**
- **Dimenticare @Override** — se sbagli firma, fai overload invece di override (nessun errore, ma non funziona come previsto)
- **Chiamare super per sbaglio** — se vuoi estendere, chiama `super.metodo()`, se vuoi sostituire completamente no

## Overload — stesso nome, parametri diversi

Metodi con lo stesso nome ma numero/tipo/ordine di parametri **diverso**.

```java
public class Calcolatrice {
    // Overload per tipo
    public int somma(int a, int b) { return a + b; }
    public double somma(double a, double b) { return a + b; }

    // Overload per numero parametri
    public int somma(int a, int b, int c) { return a + b + c; }

    // Overload per ordine
    public String somma(String a, int b) { return a + b; }
    public String somma(int a, String b) { return a + b; }
}
```

**Quando usarlo:**
- Metodi che fanno la stessa cosa ma con input diversi
- Costruttori multipli
- Versioni con valori di default

**Best Practice:** un metodo principale (con tutti i parametri) e overload che chiamano il principale con default.

```java
public void creaUtente(String nome, String email, boolean notify) { ... }
public void creaUtente(String nome, String email) {
    creaUtente(nome, email, true);  // notify = true di default
}
```

**Problema comune:** overload con tipi primitivi e wrapper può essere ambiguo:

```java
void foo(int x) { }
void foo(Integer x) { }
foo(null);  // ambiguità — Integer o int? ERRORE in compilazione!
```

## Interfacce — contratti che le classi implementano

Un'interfaccia definisce **cosa** una classe deve saper fare, non **come**.

```java
public interface Volante {
    void decolla();
    void atterra();
}

// Implementazione
public class Aereo implements Volante {
    @Override
    public void decolla() {
        System.out.println("L'aereo decolla");
    }

    @Override
    public void atterra() {
        System.out.println("L'aereo atterra");
    }
}
```

**Cosa può contenere un'interfaccia (Java 8+):**
- Costanti (`public static final` — implicitamente)
- Metodi astratti (da implementare)
- Metodi `default` (con corpo — Java 8)
- Metodi `static` (Java 8)

**Vantaggi:**
- **Implementazione multipla** — una classe può implementare più interfacce (non può estendere più classi)
- **Tipo formale** — usi l'interfaccia come tipo, la classe concreta è intercambiabile
- **Basso accoppiamento** — il codice dipende dal contratto, non dall'implementazione

```java
// Tutti i volatili possono essere trattati allo stesso modo
Volante v1 = new Aereo();
Volante v2 = new Uccello();
v1.decolla();  // non importa cosa sia, so che può decollare
```

**Differenza classe astratta vs interfaccia:**

| Aspetto | Classe Astratta | Interfaccia |
|---|---|---|
| Stato | Può avere campi di istanza | Solo costanti |
| Costruttore | Sì | No |
| Metodi concreti | Sì | Solo `default` e `static` |
| Ereditarietà | Una sola classe | Molte interfacce |
| Quando usarla | Classi imparentate con codice condiviso | Capacità trasversali (volante, confrontabile, serializzabile) |

## Confronto Override vs Overload

| | Override | Overload |
|---|---|---|
| Firma | Identica | Diversa (parametri) |
| Classe | Diversa (super → sub) | Stessa classe |
| Scopo | Sostituire comportamento | Aggiungere varianti |
| Risoluzione | Runtime (dynamic dispatch) | Compilazione |
| @Override | Obbligatorio (buona pratica) | Non si usa |

​