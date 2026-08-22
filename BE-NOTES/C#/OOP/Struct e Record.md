---
topic: "Struct e Record in C#"
nav_prev: "[[Classi Astratte.md]]"
nav_next: "[[Pattern Matching.md]]"
---

Le **struct** sono value type, le **classi** sono reference type, i **record** (C# 9+) sono un tipo nuovo che unisce immutabilità per valore a sintassi concisa. Dal C# 10 esistono `record class` (reference type) e `record struct` (value type).

## Perché esistono
Le struct servono per dati piccoli e frequenti dove la semantica per valore è naturale e la pressione sul GC va ridotta. I record nascono per eliminare il boilerplate dei DTO immutabili: `Equals`/`GetHashCode`/`ToString` generati dal compilatore, deconstruction, non-distructive mutation con `with`.

## Struct (value type)

```csharp
public struct Punto
{
    public int X { get; set; }
    public int Y { get; set; }

    public Punto(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    // readonly member — non modifica lo stato
    public readonly double DistanzaDaOrigine() => Math.Sqrt(X * X + Y * Y);
}

// Uso
Punto p1 = new(3, 4);
Punto p2 = p1;          // COPIA: p2 è indipendente
p2.X = 10;              // p1.X rimane 3
```

Le struct:
- Non supportano ereditarietà (ma implementano interfacce)
- Non possono avere costruttore senza parametri (C# < 10)
- Hanno semantica per valore (copiate su assegnamento)
- Sono allocate inline (stack o dentro un oggetto)
- Sono ideali per dati ≤ 16 byte

### readonly struct

```csharp
public readonly struct Coordinate
{
    public double Lat { get; }
    public double Lon { get; }
    
    public Coordinate(double lat, double lon)
    {
        Lat = lat;
        Lon = lon;
    }
}
```

`readonly struct` garantisce che tutti i campi siano readonly — il compilatore può ottimizzare eliminando copie difensive.

## Record (C# 9+, reference type)

```csharp
// Record: immutabile, valore per uguaglianza, deconstruct, with
public record Persona(string Nome, string Cognome, int Eta);

// Uso
var p1 = new Persona("Mario", "Rossi", 30);
var p2 = new Persona("Mario", "Rossi", 30);

Console.WriteLine(p1 == p2);   // true — uguaglianza per VALORE

// Non-distructive mutation
var p3 = p1 with { Eta = 31 };  // nuova istanza con solo Eta cambiata

// Deconstruction
var (nome, cognome, eta) = p3;
Console.WriteLine(nome);       // "Mario"
```

Il compilatore genera automaticamente:
- Costruttore primario con parametri
- Proprietà init-only (pubbliche, immutabili)
- `Equals()` / `GetHashCode()` per valore
- `Deconstruct()` per deconstruction
- `ToString()` con tutti i campi
- Operatori `==` e `!=` per valore

### Record con corpo

```csharp
public record Prodotto
{
    public string Nome { get; init; }
    public decimal Prezzo { get; init; }
    public int Quantita { get; init; }
    
    // Proprietà calcolata
    public decimal Totale => Prezzo * Quantita;
}
```

## Record struct (C# 10+, value type)

```csharp
// Record struct: immutabile + value type + uguaglianza strutturale
public readonly record struct Punto3D(double X, double Y, double Z);

// Oppure mutable (ma perché?):
public record struct PuntoMutable(double X, double Y);
```

`readonly record struct` combina i benefici dei record (uguaglianza per valore, with, deconstruction) con le performance delle struct (nessuna allocazione heap, meno GC pressure).

## Record vs class vs struct — quando usare cosa

| Scenario | Tipo consigliato | Perché |
|----------|:-:|---|
| DTO / API response / dati immutabili | `record` | Concisione, uguaglianza per valore, with |
| Entity con identità e comportamento | `class` | Reference type, identità, ereditarietà |
| Dati piccoli e frequenti (≤16 byte) | `readonly record struct` | Performance, nessuna allocazione |
| Value type matematici | `readonly struct` | Semantica per valore, performance |
| ViewModel/State mutabile | `class` | Reference type, binding UI |
| Option/Result pattern | `readonly record struct` | Immutabile, performante, non allocante |

```csharp
// Entity con identità — classe
public class Ordine
{
    public Guid Id { get; init; } = Guid.NewGuid();
    public string Cliente { get; set; }
    public decimal Totale { get; private set; }
    
    public void AggiungiVoce(decimal importo) => Totale += importo;
}

// DTO — record
public record OrdineDto(Guid Id, string Cliente, decimal Totale);

// Dati geometrici piccoli — record struct
public readonly record struct Vettore(double X, double Y, double Z);
```

## Non-distructive mutation con `with`

```csharp
public record Configurazione
{
    public string Host { get; init; } = "localhost";
    public int Porta { get; init; } = 8080;
    public bool UsaSsl { get; init; } = false;
    public string? Utente { get; init; }
}

// Copia con modifiche — senza mutare l'originale
var defaultConfig = new Configurazione();
var customConfig = defaultConfig with { Porta = 443, UsaSsl = true };
```

`with` crea una shallow copy dell'istanza e modifica solo le proprietà specificate. L'originale rimane immutato — thread-safe per definizione.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Struct grande (>16 byte)** | Performance peggiore di class | Copiata per valore su ogni passaggio | Usare class o passare `in` / `ref` |
| **Record mutable con set** | Uguaglianza per valore che cambia | Proprietà con `set` in record | Usare `init` o rendere il record class normale |
| **Uguaglianza record cambiata** | Hash set o dizionario non funziona | Mutare un record dopo averlo inserito in un set | Rendere record immutabili o non usarli in set/dizionari |
| **Struct in boxing** | Performance degradata | Assegnare struct a interfaccia | Usare generics con constraint |
| **Record.ToString() in log** | Esporre dati sensibili | ToString() mostra tutti i campi | Override ToString() o usare attributo [SensitiveData] |

## Best Practices

- **Usa `record` come default per dati** — ottenere immutabilità e uguaglianza per valore gratis
- **Usa `readonly struct` per dati piccoli** (≤16 byte) in contesti performance-critical
- **Usa `class` per entità con identità** (confronto referenziale) e comportamento
- **Usa `with` per mutation non-distruttiva** — thread-safe, predicibile
- **Non mutare record dopo averli usati come chiavi** — viola l'invariante di hash
- **Preferisci `readonly record struct`** per value type immutabili
- **Per struct grandi, passa come `in` parametro** per evitare copie
