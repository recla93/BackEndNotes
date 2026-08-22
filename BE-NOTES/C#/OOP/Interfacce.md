---
topic: "Interfacce in C#"
nav_prev: "[[Ereditarietà.md]]"
nav_next: "[[Classi Astratte.md]]"
---

Un'interfaccia in C# definisce un **contratto** — un insieme di membri (metodi, proprietà, eventi, indicizzatori) che una classe o struct deve implementare. A differenza delle classi astratte, le interfacce non contengono stato (fino a C# 8).

## Perché esistono
Le interfacce permettono di separare "cosa" un oggetto fa da "come" lo fa. Sono la base del polimorfismo per comportamento condiviso tra tipi non correlati gerarchicamente. Una classe può implementare **più interfacce** — questo è l'unico supporto per ereditarietà multipla in C#.

## Interfaccia di base

```csharp
public interface IVolante
{
    void Decolla();
    void Atterra();
}

public interface IPasseggero
{
    int Posti { get; }
}

// Implementazione multipla
public class Aereo : IVolante, IPasseggero
{
    public int Posti => 150;
    
    public void Decolla() => Console.WriteLine("Aereo decolla");
    public void Atterra() => Console.WriteLine("Aereo atterra");
}

// Uso polimorfico
IVolante volante = new Aereo();
volante.Decolla();
```

I nomi delle interfacce iniziano convenzionalmente con `I`. Una classe può implementare interfacce multiple (separate da virgole) e derivare da una sola classe base.

## Interfacce con implementazione di default (C# 8+)

## Perché esistono
Aggiungere un metodo a un'interfaccia pubblica rompe TUTTE le implementazioni esistenti. Con l'implementazione di default, si può estendere un'interfaccia senza breaking change.

```csharp
public interface ILogger
{
    void Log(string message);
    
    // Implementazione di default (C# 8+)
    void LogError(string message) => Log($"[ERRORE] {message}");
}

public class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
    // LogError è ereditata dall'implementazione default
}

// Uso
ILogger logger = new ConsoleLogger();
logger.Log("test");           // chiama ConsoleLogger.Log
logger.LogError("errore");    // chiama l'implementazione default in ILogger
```

ATTENZIONE: l'implementazione di default è chiamata solo quando l'oggetto è referenziato tramite l'interfaccia. Se la classe fornisce una propria implementazione, quella vince.

## Interfacce esplicite

## Perché esistono
Quando una classe implementa due interfacce con lo stesso metodo, serve distinguere. L'implementazione esplicita è accessibile SOLO tramite l'interfaccia, non tramite l'istanza.

```csharp
public interface IWriter
{
    void Write(string text);
}

public interface ILogger2
{
    void Write(string message);
}

public class FileLogger : IWriter, ILogger2
{
    // Implementazione esplicita per IWriter
    void IWriter.Write(string text) => File.AppendAllText("output.txt", text);
    
    // Implementazione esplicita per ILogger2
    void ILogger2.Write(string message) => File.AppendAllText("log.txt", $"[LOG] {message}");
    
    // Metodo pubblico (opzionale)
    public void Scrivi(string testo) => Console.WriteLine(testo);
}

// Uso
var logger = new FileLogger();
// logger.Write("test");   // ERRORE: ambiguo
((IWriter)logger).Write("test");     // OK: usa IWriter.Write
((ILogger2)logger).Write("test");    // OK: usa ILogger2.Write
```

L'implementazione esplicita è utile per risolvere conflitti di nome e per nascondere metodi "interni" dell'interfaccia dall'API pubblica della classe.

## Variance nelle interfacce

```csharp
// Covariante: T solo in posizione di output
public interface IProduttore<out T>
{
    T Produce();
}

// Controvariante: T solo in posizione di input
public interface IConsumatore<in T>
{
    void Consuma(T item);
}

// Uso
IProduttore<Gatto> prodGatti = new ProduttoreGatti();
IProduttore<Animale> prodAnimali = prodGatti;  // covarianza: Gatto → Animale

IConsumatore<Animale> consAnimali = new ConsumatoreAnimali();
IConsumatore<Gatto> consGatti = consAnimali;   // controvarianza: Animale → Gatto
```

## Interfacce più comuni in .NET

| Interfaccia | Scopo | Esempio di uso |
|-------------|-------|----------------|
| `IDisposable` | Rilascio risorse | `using` statement, file, connessioni |
| `IEnumerable<T>` | Iterazione | `foreach`, LINQ |
| `IComparable<T>` | Ordinamento | `Array.Sort()`, `List<T>.Sort()` |
| `IEquatable<T>` | Uguaglianza | `Dictionary<TKey, TValue>`, `HashSet<T>` |
| `ICloneable` | Clonazione (sconsigliato) | Preferire copy constructor o factory |
| `IFormattable` | Formattazione cultura-specifica | `ToString("N2", culture)` |
| `INotifyPropertyChanged` | Data binding | WPF, MAUI, Blazor |
| `IServiceProvider` | Service locator | DI container |

```csharp
// Esempio: IComparable per ordinamento custom
public class Prodotto : IComparable<Prodotto>
{
    public string Nome { get; set; }
    public decimal Prezzo { get; set; }
    
    public int CompareTo(Prodotto? other)
    {
        if (other is null) return 1;
        return Prezzo.CompareTo(other.Prezzo);  // ordina per prezzo
    }
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Esplicitare interfaccia quando non serve** | Codice verboso | Implementare esplicitamente anche se non c'è conflitto | Preferire implementazione implicita per membri principali |
| **Default interface method non chiamato** | Comportamento inaspettato | Chiamare il metodo sull'istanza invece che sull'interfaccia | Usare cast all'interfaccia per chiamare l'implementazione default |
| **Interfaccia troppo grande (ISP violato)** | Classi con metodi che non servono | Un'interfaccia "omnibus" con troppi membri | Dividere in interfacce piccole e focalizzate (Interface Segregation Principle) |
| **Implementazione esplicita + boxing** | Performance degradata | Struct che implementa interfaccia — boxing al cast | Usare generics con constraint per evitare boxing |
| **Equals/GetHashCode su interfaccia** | Dizionario non funziona | `IEquatable<T>` implementato ma `Equals(object)` no | Implementare entrambi o usare `EqualityComparer<T>.Default` |

## Best Practices

- **Segui Interface Segregation Principle** (ISP) — interfacce piccole e focalizzate (1-3 metodi)
- **Preferisci interfacce a classi astratte** quando non c'è implementazione condivisa da ereditare
- **Usa `IReadOnlyList<T>`** invece di `List<T>` come tipo di ritorno — non esporre mutabilità
- **Marca le interfacce con `[Obsolete]` invece di rompere implementazioni** quando devi deprecare
- **Non creare interfacce marker vuote** — usa attributi invece (`[Serializable]` è meglio di `ISerializable`)
- **Per factory/Dependency Injection**, definisci interfacce per i servizi, non direttamente per le classi
- **Documenta il contratto** — cosa deve fare, non come implementarlo
