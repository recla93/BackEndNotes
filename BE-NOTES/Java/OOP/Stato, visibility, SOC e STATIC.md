---
topic: "Stato, visibility, SOC e STATIC"
nav_prev: "[[Metodi.md]]"
nav_next: "[[Override, interfaces, Overload.md]]"
---

## Stato dell'Oggetto

Lo **stato** di un oggetto è l'insieme dei valori delle sue proprietà in un determinato momento.

```java
Persona p = new Persona("Mario", 30);
// Stato: { nome="Mario", eta=30 }
p.setEta(31);
// Stato: { nome="Mario", eta=31 } — lo stato è cambiato
```

Lo stato evolve tramite **metodi**. L'esterno interagisce con lo stato tramite **getter** (leggere) e **setter** (modificare).

**Principio:** l'oggetto deve essere responsabile del proprio stato.

```java
public class ContoBancario {
    private double saldo;

    public void preleva(double importo) {
        if (importo <= 0) throw new IllegalArgumentException("Importo non valido");
        if (importo > saldo) throw new IllegalStateException("Saldo insufficiente");
        this.saldo -= importo;
    }
}
```

## Incapsulamento (Encapsulation)

**Incapsulamento** = nascondere i dettagli interni, esporre solo ciò che serve.

```java
public class Persona {
    private String nome;
    private int eta;

    public String getNome() { return nome; }
    public void setNome(String nome) {
        if (nome == null || nome.isBlank())
            throw new IllegalArgumentException("Nome obbligatorio");
        this.nome = nome;
    }
}
```

**Vantaggi:**
- Validazione nei setter (controllo su cosa entra)
- Libertà di cambiare implementazione interna senza impattare i client
- Stato sempre coerente

## Separation of Concerns (SoC)

**SoC** = ogni componente ha **una singola responsabilità** ben definita.

```java
// ❌ VIOLAZIONE: una classe che fa tutto
public class PersonaService {
    public void salvaPersona(Persona p) {
        // logica business + scrittura log + invio email + accesso DB
    }
}

// ✅ CORRETTO: ogni classe ha una responsabilità
@Service
public class PersonaService { ... }
@Component
public class EmailService { ... }
@Component
public class AuditService { ... }
```

I controlli e la logica devono stare nel contesto dell'oggetto che possiede i dati, non nel chiamante.

## Modificatore Static

`static` = appartiene alla **classe**, non all'istanza:

```java
public class Contatore {
    private static int contatoreGlobale = 0;  // 1 copia per TUTTA la classe
    private int id;                            // 1 copia per OGNI oggetto

    public Contatore() {
        contatoreGlobale++;
        this.id = contatoreGlobale;
    }

    public static int getContatoreGlobale() { return contatoreGlobale; }
}

// Uso
new Contatore();  // id=1, contatoreGlobale=1
new Contatore();  // id=2, contatoreGlobale=2
Contatore.getContatoreGlobale();  // 2 — chiamato SULLA CLASSE
```

**Caratteristiche:**
- Appartiene alla classe, non all'oggetto
- Chiamabile senza istanza: `Math.max(5, 3)`
- Non ha accesso a `this`
- Non può accedere a variabili di istanza
- Non può essere override (solo nascosto)

**Quando usare:**
- Costanti (`public static final`)
- Utility methods (`Math`, `Collections`)
- Factory methods (`List.of()`)
- Contatori globali
- Metodo `main()`

## Static Final — Costanti

```java
public class Config {
    public static final int MAX_TENTATIVI = 3;
    public static final String APP_NAME = "TaskManager";
    public static final double IVA = 0.22;
}

Config.MAX_TENTATIVI  // 3
```

- `static` → una copia per tutta la classe
- `final` → non può essere riassegnata

**Convenzione:** `MAIUSCOLO_CON_UNDERSCORE` per le costanti.

## Modificatore Final

`final` impedisce la riassegnazione:

```java
// Variabile locale
final int x = 5;
// x = 6;  ERRORE

// Campo di istanza
public class Persona {
    private final long id;      // impostato nel costruttore, mai più cambiato
    private final String nome;
}

// Metodo — non può essere override
public final void metodoFinale() { ... }

// Classe — non può essere estesa
public final class String { ... }
```

**Differenza static vs final:**

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|---|---|---|---|
| Static method chiamato su istanza | Compila ma confonde: `oggetto.metodoStatico()` | Java permette la chiamata su istanza ma non è idiomatico | Chiama sempre sul tipo: `Classe.metodoStatico()` |
| Static method che accede a campo d'istanza | Errore di compilazione | Un metodo `static` non vede `this` | Rendi il metodo non statico o passa il campo come parametro |
| Modificare `final` | Errore di compilazione | Un campo `final` può essere assegnato solo una volta | Se devi modificare, togli `final`. Se è una costante, lascialo |
| Static inteso come "globale mutabile" | Race condition in contesto multi-thread | `static` non è thread-safe di default | Usa `AtomicInteger`, `synchronized`, o `ThreadLocal` per stato condiviso |
| Override di metodo statico | Il metodo chiamato è quello del tipo dichiarato, non del runtime | I metodi statici non fanno dynamic dispatch — sono nascosti (hide), non override | Non usare `@Override` su metodi statici, e non chiamarli su istanza |
| Esporre array/campo mutabile via getter | Il chiamante modifica lo stato interno | Il getter restituisce il riferimento diretto all'array | Restituisci una copia: `Arrays.copyOf()` o `Collections.unmodifiableList()` |
| Getter/setter automatici senza validazione | Stato incoerente: età negativa, nome vuoto | Non convalidi nei setter | Aggiungi validazione nei setter; se non serve validazione, valuta un record |

| | static | final |
|---|---|---|
| Cosa fa | Una copia per classe | Non può essere riassegnato |
| Per oggetto? | No (una per classe) | Sì (se campo d'istanza) |
| Combinato | `static final` = costante globale |
| Esempio | `contatoreGlobale` | `id` univoco per persona |