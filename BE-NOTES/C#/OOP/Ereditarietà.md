---
topic: "Ereditarietà in C#"
nav_prev: "[[Classi e Oggetti.md]]"
nav_next: "[[Interfacce.md]]"
---

L'ereditarietà in C# permette a una classe di derivare da un'altra, ereditandone membri e comportamento. C# supporta ereditarietà **singola** (una sola base class) ma implementazione **multipla** di interfacce.

## Perché esiste
L'ereditarietà modella relazioni "is-a" (un `Gatto` è un `Animale`) e permette il riuso del codice attraverso la gerarchia. A differenza di C++, C# vieta l'ereditarietà multipla di classi per evitare il "diamond problem" — ma la risolve con le interfacce.

## Sintassi base

```csharp
public class Animale
{
    public string Nome { get; set; }
    
    public void Respira() => Console.WriteLine($"{Nome} respira");
    
    // Virtual: può essere sovrascritta nelle classi derivate
    public virtual void Verso() => Console.WriteLine("?");
}

public class Cane : Animale  // ereditarietà singola
{
    // Override: sostituisce l'implementazione base
    public override void Verso() => Console.WriteLine("Bau!");
    
    public void Scodinzola() => Console.WriteLine($"{Nome} scodinzola");
}
```

`Cane` eredita `Nome`, `Respira()` e `Verso()`. `override` sostituisce il metodo virtuale. Le classi derivate possono aggiungere nuovi membri ma non rimuovere quelli ereditati.

## Virtual, override, new, sealed

```csharp
public class Base
{
    public virtual void Mostra() => Console.WriteLine("Base");
    public void NonVirtuale() => Console.WriteLine("Sempre Base");
}

public class Derivata : Base
{
    // override: sostituisce l'implementazione virtuale
    public override void Mostra() => Console.WriteLine("Derivata");
    
    // new: nasconde il metodo base (ATTENZIONE: polimorfismo diverso!)
    public new void NonVirtuale() => Console.WriteLine("Derivata (new)");
}

public sealed class Finale : Derivata
{
    // sealed: ferma la catena di override
    public sealed override void Mostra() => Console.WriteLine("Finale");
}

// Uso — differenza tra override e new
Base b = new Derivata();
b.Mostra();        // "Derivata" (override: polimorfismo funziona)
b.NonVirtuale();   // "Sempre Base" (new: polimorfismo NON funziona)
```

| Keyword | Effetto |
|---------|---------|
| `virtual` | Il metodo PUÒ essere sovrascritto (ma non è obbligatorio) |
| `override` | Sostituisce il metodo virtuale della base — polimorfismo funziona |
| `new` | Nasconde il metodo base — nessun polimorfismo |
| `sealed` | Impedisce ulteriore override (su classe o metodo) |
| `abstract` | Nessuna implementazione — la classe derivata DEVE implementarlo |

## Classi astratte

## Perché esistono
Una classe astratta non può essere istanziata direttamente; serve come base per classi concrete. Fornisce implementazione parziale e obbliga le derivate a implementare certi metodi.

```csharp
public abstract class Forma
{
    public string Colore { get; set; }
    
    // Metodo astratto: nessuna implementazione, le derivate DEVONO implementarlo
    public abstract double Area();
    
    // Metodo concreto: implementazione condivisa da tutte le derivate
    public virtual void Disegna() => Console.WriteLine($"Disegno forma {Colore}");
}

public class Cerchio : Forma
{
    public double Raggio { get; set; }
    
    public override double Area() => Math.PI * Raggio * Raggio;
    
    public override void Disegna() => Console.WriteLine($"Disegno cerchio di raggio {Raggio}");
}
```

Una classe astratta può avere:
- Campi, proprietà, costruttori
- Metodi concreti e astratti
- Implementazione parziale

Non può essere istanziata con `new`. Scegli abstract class quando c'è una relazione **is-a** con implementazione condivisa.

## Chiamata al costruttore base

```csharp
public class Veicolo
{
    public string Marca { get; }
    public int Anno { get; }
    
    public Veicolo(string marca, int anno)
    {
        Marca = marca;
        Anno = anno;
    }
}

public class Auto : Veicolo
{
    public int Porte { get; }
    
    public Auto(string marca, int anno, int porte) 
        : base(marca, anno)  // chiama il costruttore di Veicolo
    {
        Porte = porte;
    }
}
```

Se la classe base non ha un costruttore senza parametri, la derivata DEVE chiamare `base(...)` esplicitamente.

## Pattern sealed e divieto di ereditarietà

```csharp
// Classe sealed: non può essere usata come base
public sealed class Configurazione
{
    public string ConnectionString { get; init; }
}

// Errore: 'Derivata' cannot derive from sealed type 'Configurazione'
// public class Derivata : Configurazione { }
```

Usa `sealed` sulle classi quando:
- La classe è stata progettata per essere definitiva (es. utility class)
- Vuoi prevenire override non sicuri
- Performance: il JIT può devirtualizzare chiamate su classi sealed

## Covarianza dei tipi derivati

```csharp
// Covarianza nei tipi di ritorno (C# 9+)
public class CloneVirtual
{
    public virtual CloneVirtual Clona() => new();
}

public class CloneSpecifico : CloneVirtual
{
    public override CloneSpecifico Clona() => new();  // tipo di ritorno covariante!
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Cast non valido** | `InvalidCastException` | Assumere che un oggetto sia di un tipo derivato | Usare `is`/`as` prima del cast |
| **Chiamata virtual nel costruttore** | Bug inaspettati | Il metodo virtual chiama implementazione derivata prima che sia inizializzata | Rendere i metodi sealed o non chiamarli in costruttori |
| **new invece di override** | Polimorfismo non funziona | Nascondere il metodo invece di sovrascriverlo | Usare `override` se serve polimorfismo |
| **Base non inizializzata** | Errore di compilazione | Classe base senza costruttore predefinito | Chiamare `base(...)` esplicitamente |
| **Deep hierarchy (>3 livelli)** | Manutenibilità scarsa | Troppi livelli di astrazione | Preferire composizione su ereditarietà |
| **Manca sealed su override** | Rottura dell'invariante | Classe derivata sovrascrive comportamento critico | Sealed sull'override se il comportamento non deve cambiare |

## Best Practices

- **Preferisci composizione su ereditarietà** — "has-a" è più flessibile di "is-a"
- **Limita la profondità** — massimo 2-3 livelli di ereditarietà
- **Marca `sealed` le classi che non sono state progettate per essere estese** — documenta l'intenzione
- **Usa `virtual` solo se prevedi e supporti l'override** — non è il default
- **Non chiamare metodi virtual nel costruttore** — la classe derivata non è ancora inizializzata
- **Preferisci classi astratte per gerarchie con implementazione condivisa**, interfacce per contratti puri
- **Usa `record` invece di classi gerarchiche** per dati immutabili senza comportamento
