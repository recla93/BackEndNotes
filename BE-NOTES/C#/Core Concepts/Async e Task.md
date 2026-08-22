---
topic: "Async e Task in C#"
nav_prev: "[[Delegati ed Eventi.md]]"
nav_next: "[[Classi e Oggetti.md]]"
---

La programmazione asincrona in C# è costruita attorno ai tipi `Task` e `Task<T>`, con le keyword `async`/`await` come zucchero sintattico. Il modello è basato su **promesse**: un'operazione avviata restituisce subito un `Task` che rappresenta il lavoro futuro.

## Perché esiste
Le operazioni I/O (file, rete, database) passano la maggior parte del tempo ad aspettare — non a usare CPU. Con i thread classici, ogni operazione bloccante spreca un thread handle. `async`/`await` rilascia il thread durante l'attesa, permettendo di gestire migliaia di operazioni concorrenti con pochi thread.

## Task e Task<T>

```csharp
// Task — operazione asincrona SENZA valore di ritorno
Task ScaricaFileAsync(string url)
{
    return Task.Run(() => {
        // lavoro sincrono in un thread pool thread
        Thread.Sleep(1000);
    });
}

// Task<T> — operazione asincrona CON valore di ritorno
Task<int> ContaCaratteriAsync(string url)
{
    return Task.Run(() => {
        string contenuto = new HttpClient().GetStringAsync(url).Result; // BLOCCANTE — non fare!
        return contenuto.Length;
    });
}
```

`Task` rappresenta un'operazione in corso. `Task<T>` rappresenta un'operazione che produrrà un valore di tipo T. Lo stato interno include: completed, faulted (eccezione), cancelled.

## async / await

```csharp
// Metodo asincrono: async Task<T> per valori, async Task per void
public async Task<int> ScaricaEContaAsync(string url)
{
    using var client = new HttpClient();
    
    // await rilascia il thread corrente, lo riprende quando il task completa
    string contenuto = await client.GetStringAsync(url);
    
    // Questo codice viene eseguito quando GetStringAsync completa
    return contenuto.Length;
}

// Chiamata
int lunghezza = await ScaricaEContaAsync("https://example.com");
Console.WriteLine($"Lunghezza: {lunghezza}");
```

`await` non blocca il thread — lo sospende e registra una continuation. Quando il task completa, il thread riprende (nel SynchronizationContext originale — ad es. UI thread in WPF/Windows Forms). Il compilatore trasforma il metodo in una state machine.

## Regole fondamentali

```csharp
// ✅ CORRETTO: async Task<T>
public async Task<string> GetDataAsync()
{
    return await FetchDataAsync();
}

// ❌ SBAGLIATO: async void (tranne per event handler)
public async void OnButtonClick(object sender, EventArgs e)
{
    await DoWorkAsync();  // eccezione non catturabile!
}

// ✅ CORRETTO: event handler async void
public async void OnButtonClick(object? sender, EventArgs e)
{
    try
    {
        await DoWorkAsync();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message);  // gestione obbligatoria
    }
}

// ⚠️ NON usare .Result o .Wait() — deadlock garantito
public string GetDataBAD()
{
    return ScaricaEContaAsync("url").Result;  // DEADLOCK nei contesti UI/ASP.NET!
}
```

`async void` è solo per event handler: le eccezioni non possono essere catturate da un try-catch chiamante e crashano il processo. .Result e .Wait() causano deadlock nei SynchronizationContext a thread singolo (UI, ASP.NET Classic).

## Cancellazione con CancellationToken

```csharp
public async Task<int> ScaricaConTimeoutAsync(string url, CancellationToken ct)
{
    using var client = new HttpClient();
    var response = await client.GetAsync(url, ct);  // ct propagato
    string body = await response.Content.ReadAsStringAsync(ct);
    return body.Length;
}

// Uso con timeout
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
try
{
    int result = await ScaricaConTimeoutAsync("https://example.com", cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Operazione cancellata (timeout)");
}
```

`CancellationToken` è il modo standard per cancellare operazioni asincrone. Il `CancellationTokenSource` genera token e permette di cancellare. Propagare `ct` a tutte le chiamate asincrone interne.

## Task.WhenAll e WhenAny

## Perché esistono
Quando servono più operazioni asincrone indipendenti, è inefficiente attenderle una alla volta. `WhenAll` le esegue in parallelo; `WhenAny` reagisce alla prima completata.

```csharp
// Sequenziale (lento) — 3 secondi
var r1 = await ScaricaAsync("url1");
var r2 = await ScaricaAsync("url2");
var r3 = await ScaricaAsync("url3");

// Parallelo (veloce) — 1 secondo se ogni operazione è da 1s
var task1 = ScaricaAsync("url1");
var task2 = ScaricaAsync("url2");
var task3 = ScaricaAsync("url3");
var risultati = await Task.WhenAll(task1, task2, task3);

// WhenAny — primo che risponde
var tasks = new[] { ScaricaAsync("url1"), ScaricaAsync("url2") };
var primo = await Task.WhenAny(tasks);
Console.WriteLine($"Primo a rispondere: url{Array.IndexOf(tasks, primo) + 1}");
```

`WhenAll` lancia tutti i task concorrentemente e attende che TUTTI completino. Se **uno o più falliscono**, l'eccezione è aggregata. `WhenAny` restituisce il primo task completato — utile per timeout, cache racing (primo che risponde vince), o servizi ridondanti.

## Async con Stream

```csharp
public async Task<long> CopiaFileAsync(string source, string dest)
{
    await using var sourceStream = File.OpenRead(source);
    await using var destStream = File.Create(dest);
    await sourceStream.CopyToAsync(destStream);
    return sourceStream.Length;
}

// Lettura linee async
public async IAsyncEnumerable<string> LeggiRigheAsync(string path)
{
    using var reader = File.OpenText(path);
    string? line;
    while ((line = await reader.ReadLineAsync()) != null)
    {
        yield return line;
    }
}

// Consumo
await foreach (var riga in LeggiRigheAsync("file.txt"))
{
    Console.WriteLine(riga);
}
```

`IAsyncEnumerable<T>` (C# 8+) permette di produrre e consumare sequenze asincrone in modo lazy. `await foreach` consuma un elemento alla volta.

## Async ValueTask (prestazioni)

## Perché esiste
`Task<T>` è un reference type — allocato sull'heap. Per operazioni che spesso completano **sincronamente** (es. cache hit), l'allocazione è sprecata. `ValueTask<T>` è un value type che evita l'allocazione.

```csharp
private string? _cachedData;
private readonly HttpClient _client = new();

public async ValueTask<string> GetDataAsync()
{
    if (_cachedData != null)
    {
        return _cachedData;  // completamento sincrono — nessuna allocazione Task
    }
    
    _cachedData = await _client.GetStringAsync("https://api.example.com");
    return _cachedData;
}
```

Usa `ValueTask<T>` solo quando il tuo metodo ha alte probabilità (>50%) di completare sincronamente. La maggior parte dei metodi asincroni dovrebbe usare `Task<T>` — più semplice.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **.Result deadlock** | Applicazione bloccata | Chiamare `.Result` in contesto UI/ASP.NET | Usare `await` fino in cima; usare `ConfigureAwait(false)` se inevitabile |
| **async void** | Process crash senza catch | Eccezione in metodo async void | Usare `async Task` invece di `async void` |
| **Fire-and-forget senza catch** | Eccezione silenziosa | Avviare Task senza await né gestione errori | Catturare eccezioni con `ContinueWith` o try-catch nel lambda |
| **Non propagare CancellationToken** | Operazione non cancellabile | Creare token ma non passarlo | Passare ct a TUTTE le chiamate asincrone interne |
| **Thread pool starvation** | Performance degradata | Bloccare thread pool con Task.Run() per I/O | Usare `await` per I/O; `Task.Run` solo per CPU-bound |
| **Async in costruttore** | Costruttore non può essere async | Pattern errato | Usare factory pattern: `await CreaAsync()` statico |
| **Parallel accidental con async** | Race condition | Avviare più task async e modificarli | Usare `Parallel.ForEachAsync` (C# 10+) per CPU; `WhenAll` per I/O |

## Best Practices

- **Async all the way up** — non mescolare sync e async. Se inizi async, tutto lo stack deve essere async
- **Usa `ConfigureAwait(false)` in librerie** (non in UI) — evita di tornare al SynchronizationContext originale
- **Non usare `Task.Run` per operazioni I/O** — usa i metodi asincroni nativi (ReadAsync, WriteAsync, ecc.)
- **Propaga sempre CancellationToken** — ogni metodo async dovrebbe accettare un `CancellationToken`
- **Preferisci `Task.WhenAll` a sequenze di await** per operazioni indipendenti
- **Evita async void tranne che per event handler** — le eccezioni sono ingestibili
- **Usa IAsyncEnumerable<T> per flussi di dati** invece di materializzare tutto in memoria
- **Misura prima di ottimizzare con ValueTask** — la maggior parte dei metodi async beneficia poco del passaggio
