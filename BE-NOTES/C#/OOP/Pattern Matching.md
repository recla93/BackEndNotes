---
topic: "Pattern Matching in C#"
nav_prev: "[[Struct e Record.md]]"
nav_next: "[[OOP.md]]"
---

Il pattern matching in C# (evoluto da C# 7 a C# 11) permette di testare il **valore** e la **forma** di un'espressione, estraendo dati in modo sicuro. Non è solo type testing: si matchano proprietà, tuple, liste, posizioni, con operatori logici.

## Perché esiste
Prima del pattern matching, il testing di tipo richiedeva cast + if/else nidificati, e il testing di valore catene di `case` o `if`. Il pattern matching rende queste operazioni **dichiarative**, **leggibili** e **exhaustive** (il compilatore avverte se mancano casi).

## Type pattern

```csharp
// PRIMA: cast + if
if (obj is Persona)
{
    var p = (Persona)obj;
    Console.WriteLine(p.Nome);
}

// DOPO: type pattern
if (obj is Persona p)
{
    Console.WriteLine(p.Nome);  // p è in scope
}
```

Il type pattern dichiara e assegna la variabile in un unico passo. La variabile `p` è in scope solo dentro il blocco if — non c'è rischio di usare una variabile non inizializzata fuori dal pattern.

## Switch expression con pattern

```csharp
// Type + property + relational patterns
string Descrivi(object valore) => valore switch
{
    // Type pattern
    int => "Intero",
    string => "Stringa",
    
    // Relational pattern
    double d when d < 0 => "Negativo",

    // Property pattern
    Persona { Eta: >= 18 } => "Maggiorenne",
    Persona { Eta: < 18 } => "Minorenne",
    
    // Discard (default)
    _ => "Sconosciuto"
};
```

## Property pattern

```csharp
public record Indirizzo(string Via, string Citta, string CAP);

decimal CalcolaSpedizione(Indirizzo indirizzo) => indirizzo switch
{
    // Property pattern: match su proprietà nidificate
    { Citta: "Milano" } => 0m,              // gratis
    { Citta: "Roma" } or { Citta: "Torino" } => 5m,  // 5€
    { CAP: var cap } when cap.StartsWith("0") => 15m, // zone lontane
    _ => 10m    // default
};
```

## Positional pattern (deconstruct)

```csharp
public record Punto(int X, int Y);

string Quadrante(Punto p) => p switch
{
    (0, 0) => "Origine",
    ( > 0, > 0) => "Primo quadrante",
    ( < 0, > 0) => "Secondo quadrante",
    ( < 0, < 0) => "Terzo quadrante",
    ( > 0, < 0) => "Quarto quadrante",
    _ => "Su un asse"
};
```

## List pattern (C# 11+)

```csharp
// Pattern su array/liste
string AnalizzaArray(int[] numeri) => numeri switch
{
    [] => "Vuoto",
    [1] => "Solo 1",
    [1, 2] => "1 e 2",
    [1, 2, ..] => "Inizia con 1, 2, poi altro",
    [.., 9] => "Finisce con 9",
    [1, .. var resto] => $"Inizia con 1, poi {resto.Length} elementi",
    _ => "Altro"
};
```

Il list pattern usa `..` per lo slice — matcha zero o più elementi. `[1, 2, ..]` significa "almeno 2 elementi, i primi due sono 1 e 2".

## Operatori logici nei pattern (C# 9+)

```csharp
bool ÈGiornoFeriale(DayOfWeek giorno) => giorno switch
{
    DayOfWeek.Saturday or DayOfWeek.Sunday => false,
    _ => true
};

int Classifica(int eta, bool patente) => (eta, patente) switch
{
    ( >= 18, true) => 1,    // può guidare
    ( >= 18, false) => 2,   // deve prendere patente
    ( < 18, _) => 3,         // minorenne
};
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Discard mancante** | Avvertimento: switch expression not exhaustive | Manca caso `_` in switch expression | Aggiungere `_ => ...` sempre |
| **Ordine errato dei pattern** | Branch mai raggiunto | Pattern generale prima di uno specifico | Ordinare dal più specifico al più generale |
| **Property pattern su null** | `NullReferenceException` | Non gestire il caso null in pattern matching | Aggiungere `{}` (non-null) o `null` esplicitamente |
| **List pattern su array non init** | Compile error | `[]` non significa "null" | Controllare null prima del list pattern |
| **Pattern matching su tipo senza deconstruct** | Compile error | Usare positional pattern su tipo senza Deconstruct | Aggiungere Deconstruct o usare property pattern |

## Best Practices

- **Preferisci switch expression a if/else** per dispatch su tipo o valore — più espressiva e type-safe
- **Usa property pattern** per navigare gerarchie di dati senza cast intermedi
- **Combina pattern + `when`** per condizioni complesse — più leggibile di if annidati
- **Il compilatore verifica l'exhaustiveness** — se non copri tutti i casi, avverte
- **Usa list pattern** per navigare strutture ricorsive (es. linked list, alberi)
- **Evita pattern matching eccessivamente nidificato** — se serve >3 livelli, estrai un metodo
- **Usa discard `_` per variabili non necessarie** — segnala che il valore non serve
