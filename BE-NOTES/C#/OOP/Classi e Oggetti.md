---
topic: "Classi e Oggetti in C#"
nav_prev: "[[Core Concepts.md]]"
nav_next: "[[Ereditarietà.md]]"
---

La classe è il mattone fondamentale della programmazione orientata agli oggetti in C#. Definisce un tipo reference che incapsula dati (campi, proprietà) e comportamento (metodi, eventi).

## Perché esistono le classi?
Le classi forniscono un meccanismo per modellare entità del dominio con dati e comportamento accoppiati, supportando ereditarietà, polimorfismo e incapsulamento. A differenza delle struct (value type), le classi sono reference type con semantica di identità.

## Struttura di una classe

```csharp
public class Persona
{
    // Campi privati (incapsulamento)
    private string _nome;
    private int _eta;

    // Proprietà auto-implementata
    public string Cognome { get; set; }

    // Proprietà con validazione
    public string Nome
    {
        get => _nome;
        set => _nome = value ?? throw new ArgumentNullException(nameof(value));
    }

    // Proprietà read-only calcolata
    public string NomeCompleto => $"{Nome} {Cognome}";

    // Proprietà init-only (C# 9+) — set solo in initializer
    public Guid Id { get; init; } = Guid.NewGuid();

    // Costruttore
    public Persona(string nome, string cognome, int eta)
    {
        Nome = nome;       // passa dalla proprietà (validazione!)
        Cognome = cognome;
        _eta = eta;
    }

    // Costruttore primario (C# 12+)
    // public class Cliente(string nome, string email) : Persona(nome, "", 0) { }

    // Metodo
    public void Saluta() => Console.WriteLine($"Ciao, sono {NomeCompleto}");

    // Metodo statico
    public static Persona CreaAnonima() => new("Anonimo", "Utente", 0);
}
```

I campi privati sono la base dell'incapsulamento. Le proprietà forniscono accesso controllato con getter/setter/init. I costruttori inizializzano lo stato. I membri statici appartengono al tipo, non all'istanza.

## Proprietà: auto, calcolate, init-only

```csharp
public class Prodotto
{
    // Auto-proprietà — C# genera il campo privato
    public string Nome { get; set; }

    // Read-only (solo getter)
    public decimal PrezzoBase { get; }

    // init-only — set solo in object initializer
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;

    // Calcolata (expression-bodied)
    public decimal PrezzoIva => PrezzoBase * 1.22m;

    // Con backing field esplicito e validazione
    private int _quantita;
    public int Quantita
    {
        get => _quantita;
        set
        {
            if (value < 0) throw new ArgumentException("La quantità non può essere negativa");
            _quantita = value;
        }
    }
}

// Uso con object initializer
var prodotto = new Prodotto
{
    Nome = "Laptop",
    PrezzoBase = 999.99m,
    // CreatedAt = ...  // init-only: solo in initializer!
};
```

Le proprietà `init` (C# 9+) permettono di creare oggetti immutabili con object initializer. Dopo la costruzione, non possono più essere modificate — a differenza di `set` che permette modifica in qualsiasi momento.

## Object initializer e named arguments

```csharp
// Costruttore tradizionale
var p1 = new Persona("Mario", "Rossi", 30);

// Object initializer — più leggibile per classi con molte proprietà
var p2 = new Persona("Anna", "Bianchi", 25)
{
    // Posso impostare proprietà init-only qui
    // Id = Guid.NewGuid()  // è init, va bene
};

// Named arguments — utile per costruttori con parametri opzionali
var p3 = new Persona(nome: "Luigi", cognome: "Verdi", eta: 28);
```

## Costruttori e inizializzazione

```csharp
public class Risorsa
{
    private readonly string _nome;
    private readonly int _priorita;

    // Costruttore primario (C# 12+)
    public class Logger(string nomeFile)
    {
        private readonly string _path = Path.Combine("/logs", nomeFile);
        // _path è inizializzato prima del corpo del costruttore
    }

    // Catena di costruttori
    public Risorsa(string nome) : this(nome, 5) { }  // priorità default
    public Risorsa(string nome, int priorita)
    {
        _nome = nome ?? throw new ArgumentNullException(nameof(nome));
        _priorita = priorita;
    }
}
```

## Object disposal (IDisposable)

```csharp
public class FileManager : IDisposable
{
    private FileStream? _stream;
    private bool _disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);  // non serve finalizer
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing) _stream?.Dispose();
        _disposed = true;
    }
}

// Uso con using
using var manager = new FileManager();
// ... usi ...
// Dispose() chiamato automaticamente alla fine dello scope
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Proprietà invece di campo in ref** | Modifica non persiste | Passare proprietà come `ref` — non supportato | Usare campo o restituire nuovo oggetto |
| **Object initializer senza init** | Modifica accidentale | Proprietà con `set` invece di `init` | Usare `init` per dati immutabili |
| **Campo pubblico** | Incapsulamento rotto | `public string Nome;` (campo, non proprietà) | Usare proprietà `{ get; set; }` |
| **Costruttore virtual chiamato** | Bug in classi derivate | Chiamare metodo virtual nel costruttore | Non chiamare metodi virtual in costruttori |
| **Manca IDisposable** | Resource leak | Classe con risorse native senza Dispose | Implementare IDisposable + using |
| **Property accessor lento** | Performance inaspettata | Getter che esegue calcoli pesanti | Usare Lazy<T> o calcolare nel costruttore |

## Best Practices

- **Preferisci proprietà a campi pubblici** — l'incapsulamento è un pilastro dell'OOP
- **Usa `init` per dati che non devono cambiare** dopo la costruzione (immutabilità)
- **Proprietà calcolate > metodi GetXxx()** — più concise, accesso uniforme ai dati
- **Non lanciare eccezioni nei getter** (solo nei setter per validazione)
- **Usa `record` per DTO e dati immutabili**; `class` per entità con comportamento e identità
- **Implementa IDisposable** se la tua classe possiede risorse non gestite (file, socket, handle nativi)
- **Usa Constructor chaining** (`: this(...)`) per ridurre duplicazione
- **Preferisci `ArgumentNullException.ThrowIfNull(value)`** (C# 10+) per validazione parametri
