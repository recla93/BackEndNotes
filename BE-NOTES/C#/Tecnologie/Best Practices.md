---
topic: "Best Practices C#"
nav_prev: "[[Top-Level Statements.md]]"
nav_next: "[[Tecnologie.md]]"
---

Le best practice C# integrate con pattern di sviluppo moderni: TDD, immutabilità, programmazione funzionale, LINQ su loop, pattern matching su if nidificati. Ispirate ai principi del CLAUDE.md di citypaul/.dotfiles e alle convenzioni del team .NET.

## TDD in C# (RED-GREEN-MUTATE-KILL-REFACTOR)

## Perché TDD in C#
C# ha un ecosistema di testing maturo (xUnit, NUnit, FluentAssertions) e il compilatore cattura molti errori a compile-time. Il TDD aggiunge la sicurezza che il comportamento è corretto prima di scrivere l'implementazione.

```csharp
// RED: scrivi il test che fallisce
[Fact]
public void CalcolaTotale_ConSconto_DovrebbeApplicarePercentuale()
{
    var carrello = new Carrello();
    carrello.AggiungiProdotto("Laptop", 1000m, 1);
    
    var totale = carrello.CalcolaTotale(scontoPercentuale: 10);
    
    // FluentAssertions per leggibilità
    totale.Should().Be(900m);
}

// GREEN: implementazione minima
public class Carrello
{
    private readonly List<(string Nome, decimal Prezzo, int Qta)> _items = [];
    
    public void AggiungiProdotto(string nome, decimal prezzo, int quantita)
        => _items.Add((nome, prezzo, quantita));
    
    public decimal CalcolaTotale(decimal scontoPercentuale = 0)
    {
        var subTotale = _items.Sum(i => i.Prezzo * i.Qta);
        return subTotale * (1 - scontoPercentuale / 100);
    }
}
```

## Immutabilità per default

```csharp
// ✅ Immutabile: record con init-only
public record ConfigurazioneDb
{
    public string Host { get; init; } = "localhost";
    public int Porta { get; init; } = 5432;
    public bool UsaSsl { get; init; } = true;
}

// ❌ Mutabile: proprietà set pubbliche
public class ConfigurazioneDbMutable
{
    public string Host { get; set; }   // chiunque può cambiarlo
    public int Porta { get; set; }
}
```

L'immutabilità elimina intere classi di bug: race condition, side effect imprevisti, stato inconsistente. Usa `record`, `init`, `readonly` e `IReadOnlyList<T>`.

## LINQ su loop

```csharp
// ❌ Loop imperativo (mutabile, verbose)
var risultato = new List<string>();
foreach (var p in prodotti)
{
    if (p.Prezzo > 50 && p.Disponibile)
    {
        risultato.Add(p.Nome.ToUpper());
    }
}
risultato.Sort();

// ✅ LINQ (dichiarativo, immutabile, conciso)
var risultato = prodotti
    .Where(p => p.Prezzo > 50 && p.Disponibile)
    .Select(p => p.Nome.ToUpper())
    .OrderBy(n => n)
    .ToList();
```

LINQ non è solo più corto — è **dichiarativo**: dici *cosa* vuoi, non *come* ottenerlo. Non modifica le collezioni sorgenti (immutabilità).

## Pattern matching su if/else nidificati

```csharp
// ❌ If/else nidificati (complesso, fragile)
string GetCategoria(Prodotto p)
{
    if (p.Prezzo > 1000)
    {
        if (p.Quantita > 10) return "Premium Bulk";
        else return "Premium";
    }
    else if (p.Prezzo > 100)
    {
        if (p.Disponibile) return "Standard";
        else return "Standard (non disp.)";
    }
    else return "Economy";
}

// ✅ Switch expression (dichiarativo, exhaustive)
string GetCategoria(Prodotto p) => p switch
{
    { Prezzo: > 1000, Quantita: > 10 } => "Premium Bulk",
    { Prezzo: > 1000 } => "Premium",
    { Prezzo: > 100, Disponibile: true } => "Standard",
    { Prezzo: > 100 } => "Standard (non disp.)",
    _ => "Economy"
};
```

## Pure functions e nessun side effect

```csharp
// ❌ Impura: modifica stato esterno + output non deterministico
public class OrderProcessor
{
    private decimal _totale;
    public void Processa(Ordine ordine)
    {
        _totale += ordine.Totale;          // side effect su stato
        Log("Ordine processato");           // side effect su I/O
        EmailSender.Invia(ordine);          // side effect su sistema esterno
    }
}

// ✅ Pura: input → output, nessun side effect (o isolati)
public class CalcolatoreTotale
{
    public static decimal Calcola(Ordine ordine, decimal iva)
        => ordine.Voci.Sum(v => v.Prezzo * v.Quantita) * (1 + iva);
}
```

Le pure function sono facili da testare (stesso input = stesso output), composabili, e thread-safe. Isola gli effetti collaterali (I/O, email, log) ai confini del sistema.

## Gestione errori con Result pattern

```csharp
// ❌ Eccezioni per controllo di flusso
public async Task<Utente> GetUtenteAsync(int id)
{
    try
    {
        return await _db.Utenti.SingleAsync(u => u.Id == id);  // eccezione se non trovato!
    }
    catch (InvalidOperationException)
    {
        throw new UtenteNonTrovatoException(id);
    }
}

// ✅ Result pattern (valori di errore, non eccezioni)
public record Result<T>
{
    public T? Value { get; init; }
    public string? Error { get; init; }
    public bool IsSuccess => Error is null;
    
    public static Result<T> Ok(T value) => new() { Value = value };
    public static Result<T> Fail(string error) => new() { Error = error };
}

public async Task<Result<Utente>> GetUtenteAsync(int id)
{
    var utente = await _db.Utenti.FirstOrDefaultAsync(u => u.Id == id);
    return utente is not null
        ? Result<Utente>.Ok(utente)
        : Result<Utente>.Fail($"Utente {id} non trovato");
}
```

## Factory functions per test data

```csharp
// ❌ Setup ripetuto e mutabile
[Fact]
public void TestUno()
{
    var prodotti = new List<Prodotto>();  // setup inline in ogni test
    prodotti.Add(new Prodotto { Nome = "A", Prezzo = 10 });
    // ...
}

// ✅ Factory function, immutabile
public static class ProdottoFactory
{
    public static Prodotto Crea(string? nome = null, decimal? prezzo = null)
        => new()
        {
            Nome = nome ?? "Default",
            Prezzo = prezzo ?? 10m
        };
}

[Fact]
public void TestUno()
{
    var prodotti = new[] { ProdottoFactory.Crea(nome: "A", prezzo: 10) };
    // ...
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Mock di tutto** | Test fragili | Mock anche di classi interne | Testare comportamento, non implementazione |
| **Eccezioni per flusso normale** | Performance degradata | try/catch per "non trovato" | Usare Result pattern o TryGet pattern |
| **Side effect in LINQ** | Bug imprevedibili | .Select() con azioni mutanti | LINQ è per trasformazioni pure |
| **Campo pubblico** | Incapsulamento rotto | `public string Nome;` | `public string Nome { get; set; }` |
| **Stato mutabile condiviso** | Race condition | Lista statica modificata da thread multipli | Usare ImmutableList<T> o ConcurrentCollection |
| **await in Select()** | Esecuzione parallela inaspettata | `items.Select(async x => await Foo(x))` | Usare `Task.WhenAll(items.Select(Foo))` |

## Riferimenti

- [Microsoft .NET Coding Conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [CLAUDE.md — citypaul/.dotfiles](https://github.com/citypaul/.dotfiles) (principi TDD, immutabilità, FP)
- [xUnit Documentation](https://xunit.net/docs/getting-started/netcore/visual-studio)
- [FluentAssertions](https://fluentassertions.com/introduction)
