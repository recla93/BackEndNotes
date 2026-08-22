---
topic: "Delegati ed Eventi in C#"
nav_prev: "[[LINQ.md]]"
nav_next: "[[Async e Task.md]]"
---

Delegati ed eventi sono il meccanismo di callback type-safe di C#. Un delegato è un tipo che incapsula un riferimento a un metodo; un evento è un wrapper che espone un delegato solo per subscribe/unsubscribe, proteggendo l'incapsulamento.

## Perché esistono
Prima dei delegati, i callback in C/C++ usavano function pointer — non type-safe, non object-oriented. I delegati sono **reference type** che sanno quale metodo chiamare e su quale istanza, con verifica a compile-time della firma.

## Delegato come tipo

```csharp
// Dichiarazione di un tipo delegato
public delegate void LogHandler(string messaggio);

// Metodi che corrispondono alla firma
public static void LogSuConsole(string msg) => Console.WriteLine($"[CONSOLE] {msg}");
public static void LogSuFile(string msg) => File.AppendAllText("log.txt", msg);

// Uso
LogHandler logger = LogSuConsole;
logger("Applicazione avviata");   // chiama LogSuConsole

logger = LogSuFile;
logger("Applicazione avviata");   // chiama LogSuFile
```

Un delegato è un tipo che definisce una firma (parametri + tipo di ritorno). Solo metodi con firma identica possono essere assegnati. Il delegato sa quale oggetto è il target (per metodi di istanza) — chiamata object-oriented sicura.

## Multicast delegate

## Perché esistono
Un delegato in C# è un **multicast**: può referenziare più metodi, chiamati in sequenza quando il delegato viene invocato. Questo è diverso dai function pointer in C, che puntano a un solo metodo.

```csharp
LogHandler logger = LogSuConsole;
logger += LogSuFile;       // aggiunge un altro target
logger("Test");            // chiama ENTRAMBI: console + file

logger -= LogSuConsole;    // rimuove LogSuConsole
logger("Solo su file");    // chiama solo LogSuFile
```

`+=` e `-=` sono operatori che aggiungono/rimuovono metodi dalla invocation list. L'invocazione avviene in ordine di aggiunta. Se un metodo nella catena lancia un'eccezione, i successivi NON vengono eseguiti.

## Action, Func, Predicate

## Perché esistono
La maggior parte dei delegati ha una firma generica — non serve dichiarare tipi delegato custom. .NET fornisce delegati generici predefiniti.

```csharp
// Action: non restituisce valore (void), fino a 16 parametri
Action saluta = () => Console.WriteLine("Ciao");
Action<string> salutaConNome = (nome) => Console.WriteLine($"Ciao {nome}");

// Func: restituisce valore, ultimo parametro è il return type
Func<int, int, int> somma = (a, b) => a + b;
Func<int, bool> èPari = (n) => n % 2 == 0;
var risultato = somma(3, 4);         // 7

// Predicate: restituisce bool, equivalente a Func<T, bool>
Predicate<int> èPositivo = (n) => n > 0;
```

Usa sempre `Action`/`Func` invece di dichiarare delegati custom. Dichiarali custom solo se:
1. Il nome del delegato ha significato semantico nel dominio
2. Devi usare `ref`/`out` nei parametri (non supportati da generici)

## Lambda expressions

## Perché esistono
Le lambda sono la sintassi concisa per creare delegati inline. Introdotte in C# 3.0 insieme a LINQ, sono onnipresenti nel codice C# moderno.

```csharp
// Lambda completa: parametri + corpo { }
Func<int, int> quadrato = (int x) =>
{
    return x * x;
};

// Lambda semplificata: tipo inferito, espressione singola
Func<int, int> quadratoBrief = x => x * x;

// Lambda con più parametri
Func<int, int, int> sommaBrief = (a, b) => a + b;

// Lambda senza parametri
Action ciao = () => Console.WriteLine("Ciao");

// Lambda con discards (C# 9+)
Func<int, int, int> sommaDiscard = (_, _) => 0;  // ignora entrambi

// Uso con LINQ
int[] numeri = { 1, 2, 3, 4, 5 };
var pari = numeri.Where(n => n % 2 == 0).ToList();
```

La lambda cattura le variabili dello scope esterno (closure). Il compilatore genera una classe per contenere le variabili catturate. **ATTENZIONE**: la cattura estende la vita delle variabili — possibile memory leak in scenari di long-lived delegate.

## Eventi

## Perché esistono
Un evento è un wrapper su un delegato che impedisce chiamate dirette dall'esterno. Senza eventi, un delegato pubblico esposto come campo potrebbe essere invocato o resettato da qualsiasi chiamante — violando l'encapsulation.

```csharp
// Dichiarazione
public class Button
{
    // EventHandler è il delegato standard per eventi in .NET
    // sender = chi ha sollevato l'evento, args = dati
    public event EventHandler? Clicked;  // con ? per nullable reference types

    public void SimulateClick()
    {
        // Pattern standard: verifica null + invocazione thread-safe con copia
        var handler = Clicked;
        handler?.Invoke(this, EventArgs.Empty);
    }
}

// Sottoscrizione
var button = new Button();
button.Clicked += (sender, args) => Console.WriteLine("Button clicked!");

// NON si può fare (compile error):
// button.Clicked = null;           // non puoi resettare
// button.Clicked.Invoke(...);       // non puoi invocare dall'esterno
```

Ogni evento in .NET segue la convenzione: primo parametro `object? sender`, secondo un tipo che deriva da `EventArgs`. La copia in variabile locale (`var handler = Clicked`) prima dell'invocazione è un pattern thread-safe: anche se un altro thread si disiscrive, la copia locale rimane valida.

## Pattern EventHandler moderno (C# 6+)

```csharp
// Evento con dati custom
public class OrderEventArgs : EventArgs
{
    public int OrderId { get; init; }
    public decimal Total { get; init; }
}

public class OrderService
{
    // EventHandler<TEventArgs> generico
    public event EventHandler<OrderEventArgs>? OrderCompleted;

    public void CompleteOrder(int id, decimal total)
    {
        // Elaborazione...
        OrderCompleted?.Invoke(this, new OrderEventArgs { OrderId = id, Total = total });
    }
}
```

## Closure e cattura di variabili

```csharp
// ATTENZIONE: tutte le lambda catturano la STESSA variabile i
List<Action> azioni = [];
for (int i = 0; i < 5; i++)
{
    azioni.Add(() => Console.WriteLine(i));  // cattura i, non il valore!
}
foreach (var azione in azioni)
{
    azione();  // stampa "5" cinque volte!
}

// FIX: catturare una copia locale
List<Action> azioniCorrette = [];
for (int i = 0; i < 5; i++)
{
    int copia = i;  // nuova variabile per iterazione
    azioniCorrette.Add(() => Console.WriteLine(copia));
}
foreach (var azione in azioniCorrette)
{
    azione();  // stampa "0, 1, 2, 3, 4" ✓
}
```

In C# 5+ il ciclo `foreach` non ha questo problema (la variabile è scoped all'iterazione). Il ciclo `for` classico sì. Causa storica di bug sottili.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **EventHandler memory leak** | GC non rilascia subscriber | Sottoscrivere evento senza mai disiscriversi | Usare `WeakEvent` pattern o disiscriversi con `-=` |
| **NullReference su evento** | `NullReferenceException` su invocazione | Invocare evento senza controllare null | Usare `Evento?.Invoke(...)` (C# 6+) |
| **Closure in loop for** | Tutte le lambda usano l'ultimo valore | Variabile del loop catturata per riferimento | Copiare in variabile locale per iterazione |
| **Evento resettato** | Solo l'ultimo subscriber riceve notifica | Usare `=` invece di `+=` | Usare sempre `+=` per sottoscrivere, mai `=` |
| **Modifica lista subscriber durante invocazione** | Eccezione o comportamento indefinito | += o -= durante invocazione | Copiare la lista prima di iterare |
| **Eccezione non gestita in subscriber** | I subscriber successivi non vengono chiamati | Un subscriber lancia eccezione | Ogni subscriber dovrebbe gestire le proprie eccezioni |

## Best Practices

- **Preferisci `EventHandler<T>`** invece di delegati custom per eventi — convenzione .NET standard
- **Usa `Action`/`Func`** per callback generici; riserva `EventHandler` per eventi UI o di sistema
- **Disiscrivi sempre** gli eventi quando il subscriber non serve più (es. `Dispose()`) — evita memory leak
- **Controlla null** con `?.Invoke(...)` prima di sollevare un evento
- **Non usare lambda per eventi che devono essere disiscritti** — non puoi fare `-=` su una lambda anonima
- **Per metodi di callback**, preferisci parametri `Action<T>` a interfacce con un solo metodo (più lightweight)
- **Thread safety**: se l'evento può essere sollevato da thread multipli, copia il delegato in locale prima di invocare
