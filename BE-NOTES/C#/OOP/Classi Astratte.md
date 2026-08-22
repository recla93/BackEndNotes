---
topic: "Classi Astratte in C#"
nav_prev: "[[Interfacce.md]]"
nav_next: "[[Struct e Record.md]]"
---

Una classe astratta è una classe che non può essere istanziata direttamente, progettata per essere solo una base per classi derivate. Combina implementazione concreta e metodi astratti che le derivate devono implementare.

## Perché esiste
Le classi astratte colmano il gap tra interfaccia pura (nessuna implementazione) e classe concreta (tutta implementazione). Servono quando una gerarchia condivide **implementazione parziale** — logica comune che le derivate non devono riscrivere, più metodi che ogni derivata deve personalizzare.

## Differenza chiave: abstract class vs interfaccia

| Caratteristica | abstract class | interfaccia |
|---------------|:-:|:-:|
| Stato (campi) | ✅ Sì | ❌ No |
| Costruttori | ✅ Sì | ❌ No |
| Implementazione metodi | ✅ Sì (parziale) | ✅ Default (C# 8+) |
| Ereditarietà multipla | ❌ Una sola base | ✅ Multipla |
| Access modifier | ✅ Tutti | ❌ Tutto public |
| Static members | ✅ Sì | ✅ (C# 11+) |
| Quando usare | Relazione **is-a** con implementazione condivisa | Contratto **can-do** tra tipi non correlati |

```csharp
// Quando usare abstract class — c'è implementazione condivisa
public abstract class Database
{
    protected string _connectionString;
    
    protected Database(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    // Implementazione concreta (condivisa da TUTTE le derivate)
    public void Connect()
    {
        Console.WriteLine($"Connessione a {_connectionString}");
    }
    
    // Metodo astratto (specifico per ogni derivata)
    public abstract string ExecuteQuery(string sql);
}

// Quando usare interfaccia — solo contratto
public interface IExportable
{
    string Export();
}
```

## Pattern Template Method

## Perché esiste
Le classi astratte sono perfette per il **Template Method pattern**: definisci lo scheletro di un algoritmo nella classe base e lascia che le derivate implementino i passaggi variabili.

```csharp
public abstract class ReportGenerator
{
    // Template method — definisce la struttura dell'algoritmo
    public string GenerateReport()
    {
        var data = FetchData();
        var processed = ProcessData(data);
        var formatted = FormatReport(processed);
        SaveReport(formatted);
        return formatted;
    }

    // Passaggi concreti (condivisi)
    protected virtual string FetchData() => "Dati grezzi";
    
    // Passaggi astratti (obbligatori per le derivate)
    protected abstract string ProcessData(string data);
    protected abstract string FormatReport(string data);
    
    // Hook — opzionale, default vuoto
    protected virtual void SaveReport(string report) { }
}

public class PdfReport : ReportGenerator
{
    protected override string ProcessData(string data)
        => $"PDF: {data} processato";
    
    protected override string FormatReport(string data)
        => $"<pdf>{data}</pdf>";
    
    protected override void SaveReport(string report)
        => File.WriteAllText("report.pdf", report);
}
```

## Classi astratte vs sealed

```csharp
public abstract class Base
{
    public abstract void Obbligatorio();
    public virtual void Opzionale() { }
    public sealed override string ToString() => "Non sovrascrivibile";  // ferma override
}

public sealed class Concrete : Base
{
    public override void Obbligatorio() { }
    
    // public override string ToString() ... // ERRORE: sealed!
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Abstract class come interfaccia** | Campi inutilizzati, zero implementazione concreta | Usare abstract class per un puro contratto | Usare interfaccia invece |
| **Metodo virtual invece di abstract** | Base dimenticata che non fa nulla | Metodo con corpo vuoto `{ }` | Usare `abstract` per forzare implementazione |
| **Troppi livelli astratti** | Codice difficile da seguire | 4+ livelli di ereditarietà astratta | Preferire composizione o ridurre gerarchia |
| **Abstract class con zero campi** | Segnala dubbio design | Solo metodi astratti, nessuno stato | È probabilmente un'interfaccia mascherata |
| **Costruttore non protetto** | Istanziazione accidentale? | Costruttore public in abstract class | Usare `protected` per i costruttori |

## Best Practices

- **Preferisci interfaccia** se non c'è stato condiviso da ereditare
- **Usa abstract class** quando c'è logica comune (es. connection management, template method)
- **Mantieni shallow hierarchy** — massimo 2-3 livelli
- **Usa `protected` per costruttori** — solo le derivate devono poter chiamare base()
- **Documenta il contratto** delle classi astratte con XML doc comments — spiega cosa ogni metodo astratto DEVE fare
- **Combina abstract class + interfacce** quando serve: una classe astratta implementa interfacce e fornisce implementazione di base
