# Enum

## Cos'è un Enum?

Un **enum** è un insieme di **costanti predefinite**. Ogni valore è un'istanza unica (singleton) della classe enum.

```java
public enum Colore {
    ROSSO,
    VERDE,
    BLU
}

// Uso
Colore c = Colore.ROSSO;  // unica istanza
```

## Enum con Proprietà e Metodi

```java
public enum Giorno {
    LUNEDI(1, "Lunedì"),
    MARTEDI(2, "Martedì"),
    MERCOLEDI(3, "Mercoledì"),
    GIOVEDI(4, "Giovedì"),
    VENERDI(5, "Venerdì"),
    SABATO(6, "Sabato"),
    DOMENICA(7, "Domenica");

    private int numero;
    private String nome;

    // Costruttore (implicitamente privato)
    Giorno(int numero, String nome) {
        this.numero = numero;
        this.nome = nome;
    }

    public int getNumero() { return numero; }
    public String getNome() { return nome; }
}
```

## Metodi built-in degli Enum

```java
Giorno g = Giorno.LUNEDI;

g.name();                           // "LUNEDI" — nome della costante
g.ordinal();                        // 0 — posizione nell'ordine di dichiarazione
Giorno.valueOf("LUNEDI");           // Giorno.LUNEDI — da stringa a enum
Giorno.values();                    // [LUNEDI, MARTEDI, ..., DOMENICA]
Giorno.valueOf("lunedì");           // IllegalArgumentException! Case-sensitive!
```

## Switch con Enum — sempre completo

```java
public String tipoGiorno(Giorno g) {
    return switch(g) {
        case LUNEDI, MARTEDI, MERCOLEDI, GIOVEDI, VENERDI -> "Lavorativo";
        case SABATO, DOMENICA -> "Festivo";
        // Non serve default — tutti i casi coperti
    };
}
```

**Vantaggio:** il compilatore avvisa se aggiungi un nuovo valore enum e non aggiorni tutti gli switch.

## Enum con metodi astratti

Ogni costante può avere comportamento diverso:

```java
public enum Operazione {
    SOMMA {
        public double applica(double a, double b) { return a + b; }
    },
    SOTTRAZIONE {
        public double applica(double a, double b) { return a - b; }
    },
    MOLTIPLICAZIONE {
        public double applica(double a, double b) { return a * b; }
    };

    public abstract double applica(double a, double b);
}

// Uso
double r = Operazione.SOMMA.applica(5, 3);  // 8.0
```

## EnumMap e EnumSet

Collezioni ottimizzate per enum:

```java
// EnumMap — come HashMap ma più veloce (array indicizzato per ordinal)
EnumMap<Giorno, String> orari = new EnumMap<>(Giorno.class);
orari.put(Giorno.LUNEDI, "09:00-18:00");

// EnumSet — come Set ma più veloce (bit vector)
EnumSet<Giorno> weekend = EnumSet.of(Giorno.SABATO, Giorno.DOMENICA);
EnumSet<Giorno> lavorativi = EnumSet.range(Giorno.LUNEDI, Giorno.VENERDI);
```

## Quando usare enum

| Scenario | Soluzione |
|---|---|
| Valori fissi noti in compilazione | `enum` |
| Codici stato (ORDER_PENDING, SHIPPED, DELIVERED) | `enum` |
| Ruoli utente (ADMIN, USER, MODERATOR) | `enum` |
| Valori che cambiano spesso o vengono dal DB | Tabella DB, non enum |

## Vantaggi

- **Type safety** — il compilatore verifica i valori, niente stringhe magiche
- **Completezza switch** — se aggiungi un valore, il compilatore avvisa
- **Metodi e campi** — più potente di una semplice costante
- **Iterabile** — `values()` per scorrere tutti
- **Singleton** — ogni valore è unico (sicuro per `==`)

## Problemi comuni

| Problema | Esempio | Soluzione |
|---|---|---|
| **Case-sensitive** | `valueOf("lunedi")` lancia eccezione | Usa `.toUpperCase()` o confronto custom |
| **Serializzazione** | Aggiungi/rimuovi valori in produzione | Nuovo valore = problema se serializzato |
| **Troppi valori** | 100+ costanti in un enum | Probabilmente serve una tabella DB |
| **equals vs ==** | `g.equals(Giorno.LUNEDI)` | Usa `==` — gli enum sono singleton |
| **Ordinal non stabile** | `ordinal()` cambia se riordini | Non usare `ordinal()` per persistenza |
