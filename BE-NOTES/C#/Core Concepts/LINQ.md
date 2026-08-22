---
topic: "LINQ — Language Integrated Query"
nav_prev: "[[Generics.md]]"
nav_next: "[[Delegati ed Eventi.md]]"
---

LINQ (Language Integrated Query) è un insieme di operatori per eseguire query su qualsiasi fonte dati che implementi `IEnumerable<T>` o `IQueryable<T>`. Introdotto in C# 3.0, unifica la sintassi delle query su oggetti, database (LINQ to SQL, EF Core), XML e altro.

## Perché esiste
Prima di LINQ, ogni fonte dati aveva la sua sintassi di query (SQL per database, XPath per XML, loop + if per collezioni). LINQ fornisce un vocabolario **unico e type-safe** per tutte le fonti, con verifica a compile-time e IntelliSense.

## Sintassi method-based vs query-based

## Come funziona
LINQ ha due sintassi equivalenti. Quella **method-based** usa metodi di estensione su `IEnumerable<T>` e lambda. Quella **query-based** è zucchero sintattico che il compilatore traduce in chiamate ai metodi.

```csharp
int[] numeri = { 5, 2, 8, 3, 1, 9, 4, 7, 6 };

// Method syntax (più comune in produzione)
var pariMetodo = numeri
    .Where(n => n % 2 == 0)          // filtra
    .OrderByDescending(n => n)       // ordina
    .Select(n => n * 2)              // trasforma
    .ToList();                       // materializza: { 16, 12, 8, 4 }

// Query syntax (simile a SQL)
var pariQuery = (from n in numeri
                 where n % 2 == 0
                 orderby n descending
                 select n * 2).ToList();
```

La query syntax è più familiare a chi viene da SQL, ma ha meno operatori (manca `First`, `Take`, `Any`, ecc.). La method syntax è **onnicomprensiva** — ogni operatore LINQ è un metodo. Preferisci method syntax in produzione.

## Lazy evaluation (deferred execution)

## Perché è importante
LINQ è **lazy**: la query non viene eseguita finché non si itera o si chiama un operatore di materializzazione. Questo permette di comporre query senza eseguirle fino al momento necessario.

```csharp
var numeri = new List<int> { 1, 2, 3, 4, 5 };

// La query è solo DESCRITTA, non eseguita
var query = numeri.Where(n =>
{
    Console.WriteLine($"Filtrando {n}");
    return n > 2;
});

Console.WriteLine("Query non eseguita — niente output");

// La materializzazione forza l'esecuzione
var risultato = query.ToList();  // qui stampa: "Filtrando 1, 2, 3, 4, 5"

// ATTENZIONE: l'esecuzione differita può causare sorprese con closure e variabili mutate
var soglia = 3;
var query2 = numeri.Where(n => n > soglia);
soglia = 5;  // la query userà 5, non 3!
var res = query2.ToList();  // risultato: solo { } (nessun numero > 5)
```

La deferred execution è potente ma insidiosa: se la variabile catturata in una lambda cambia prima della materializzazione, il risultato cambia. **Materializza appena possibile** se la fonte dati o le variabili catturate possono cambiare.

## Principali operatori LINQ

### Filtraggio e proiezione
```csharp
int[] numeri = { 1, 2, 3, 4, 5, 6 };

// Where — filtra
var pari = numeri.Where(n => n % 2 == 0);          // { 2, 4, 6 }

// Select — proietta (trasforma ogni elemento)
var quadrati = numeri.Select(n => n * n);          // { 1, 4, 9, 16, 25, 36 }

// SelectMany — appiattisce gerarchie
var frasi = new[] { "Ciao mondo", "LINQ è bello" };
var parole = frasi.SelectMany(f => f.Split(' ')); // { "Ciao", "mondo", "LINQ", "è", "bello" }
```

### Ordinamento
```csharp
// OrderBy / ThenBy (con discendenti)
var ordinato = numeri
    .OrderBy(n => n % 2)        // prima pari (0), poi dispari (1)
    .ThenByDescending(n => n);  // poi in ordine discendente
```

### Operatori di insieme
```csharp
int[] a = { 1, 2, 3, 4 };
int[] b = { 3, 4, 5, 6 };

var unione = a.Union(b);            // { 1, 2, 3, 4, 5, 6 }
var intersezione = a.Intersect(b);  // { 3, 4 }
var differenza = a.Except(b);       // { 1, 2 }
```

### Aggregazione
```csharp
var somma = numeri.Sum();          // 21
var media = numeri.Average();      // 3.5
var minimo = numeri.Min();         // 1
var massimo = numeri.Max();        // 6
var conteggio = numeri.Count();    // 6

// Aggregate — operatore generico
var prodotto = numeri.Aggregate((acc, n) => acc * n);  // 720
```

### Quantificatori e ricerca
```csharp
bool tuttiPositivi = numeri.All(n => n > 0);    // true
bool qualchePari = numeri.Any(n => n % 2 == 0); // true
bool contiene3 = numeri.Contains(3);            // true

var primo = numeri.First();             // 1 — eccezione se vuoto!
var primoODef = numeri.FirstOrDefault(); // 1 — default(0) se vuoto
var singolo = numeri.Single(n => n == 3); // 3 — eccezione se non esattamente 1!
var singoloODef = numeri.SingleOrDefault(n => n == 99); // 0 — default se non esattamente 1
```

`First()` lancia eccezione se la sequenza è vuota. `FirstOrDefault()` restituisce il valore di default. `Single()` è più restrittivo: lancia eccezione se non c'è **esattamente** un match. Usalo quando l'invariante logico richiede unicità.

### Raggruppamento e join
```csharp
var persone = new[]
{
    new { Nome = "Mario", Citta = "Roma" },
    new { Nome = "Anna", Citta = "Milano" },
    new { Nome = "Luigi", Citta = "Roma" },
};

// GroupBy
var perCitta = persone.GroupBy(p => p.Citta);
foreach (var gruppo in perCitta)
{
    Console.WriteLine($"{gruppo.Key}: {string.Join(", ", gruppo.Select(p => p.Nome))}");
}

// GroupBy + aggregazione
var conteggi = persone
    .GroupBy(p => p.Citta)
    .Select(g => new { Citta = g.Key, Count = g.Count() });
```

## IQueryable<T> e LINQ to EF Core

## Perché esiste
`IQueryable<T>` permette di tradurre la query LINQ in SQL (o altra sintassi nativa), eseguendo il filtro e la proiezione **sul database** invece che in memoria.

```csharp
using var db = new AppDbContext();

// IQueryable — la query NON è ancora stata eseguita
IQueryable<Prodotto> query = db.Prodotti
    .Where(p => p.Prezzo > 50)
    .OrderBy(p => p.Nome);

// Qui EF Core traduce in SQL:
// SELECT * FROM Prodotti WHERE Prezzo > 50 ORDER BY Nome
var prodotti = query.ToList();  // esecuzione sul database
```

Non confondere `IEnumerable<T>` (lazy evaluation in memoria) con `IQueryable<T>` (traduzione in SQL). Una query `IQueryable` portata in memoria con `.ToList()` prematuro perde ogni capacità di filtraggio lato database.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Multiple enumeration** | Itera la stessa query più volte | Chiamare `.Count()` e poi `.ToList()` sulla stessa query | Materializzare con `.ToList()` subito |
| **Captured variable mutation** | Risultato inaspettato | Variabile catturata in Where cambia prima della materializzazione | Materializzare subito o copiare la variabile |
| **N+1 queries** | 100 query per 100 record | EF Core naviga proprietà di navigazione una per volta | Usare `.Include()` per eager loading |
| **Select N + 1 su oggetti** | Query lenta | `from p in persone select p.Ordini.Count` | Rivedere la query o materializzare prima |
| **First vs FirstOrDefault** | `InvalidOperationException` | Usare `First()` su sequenza vuota | Usare `FirstOrDefault()` se vuoto è un caso possibile |
| **ToList prematuro su IQueryable** | Enormi trasferimenti memoria | `context.Set<T>().ToList().Where(...)` | Non materializzare prima del filtro |
| **Where dopo ToList su DB** | Filtro in memoria invece che SQL | `context.Set<T>().ToList().Where(predicate)` | Mettere Where PRIMA di ToList() |

## Best Practices

- **Preferisci method syntax** — più espressiva, supporta tutti gli operatori
- **Materializza appena la logica lo richiede** — deferred execution è utile ma non abusarne
- **Usa `ToList()` per materializzare**, `ToArray()` solo se serve un array, `ToDictionary()` per lookup
- **Per IQueryable**, filtra il più a monte possibile — lascia che il DB faccia il lavoro pesante
- **Usa `Any()` invece di `Count() > 0`** — più efficiente (si ferma al primo match)
- **Non usare `Single()` se non serve l'invariante di unicità** — preferisci `FirstOrDefault()`
- **Per LINQ su oggetti**, usa `Select` per trasformare, non per side effect — viola la purezza funzionale
- **LINQ + immutabilità**: i metodi LINQ non modificano mai le collezioni sorgenti (tranne che in casi rari documentati)
