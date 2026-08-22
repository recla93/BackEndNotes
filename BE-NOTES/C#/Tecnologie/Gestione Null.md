---
topic: "Gestione Null in C#"
nav_prev: "[[OOP.md]]"
nav_next: "[[Top-Level Statements.md]]"
---

La gestione del null in C# è stata radicalmente rivoluzionata con l'introduzione dei **Nullable Reference Types** (C# 8+) e una serie di operatori che rendono il codice null-safe in modo conciso.

## Perché esiste
Null è stato definito "the billion-dollar mistake" da Tony Hoare. I nullable reference types permettono al compilatore di distinguere tra variabili che **possono** essere null e variabili che **non devono** essere null, a compile-time.

## Nullable Reference Types (C# 8+)

## Come funziona
Con `<Nullable>enable</Nullable>` nel progetto, il compilatore analizza i flussi:
- `string` = non-nullable (il compilatore avverte se può essere null)
- `string?` = nullable (il compilatore avverte se non controlli null)

Il compilatore traccia il **null state** di ogni variabile attraverso il flusso del codice (null-state analysis).

```csharp
#nullable enable

string nome = "Mario";
string? cognome = null;   // OK: dichiarato nullable

int lunghezza = nome.Length;        // OK: nome è non-null
int? lunghezza2 = cognome?.Length;  // OK: ?. propaga null

// Avvertimento: possible null reference return
string Restituisci() => cognome;    // warning! cognome è string?

// Fix: controllare null
string RestituisciSicuro() => cognome ?? "Default";
```

## Operatori null-safe

### Null-conditional operator (`?.`)
```csharp
// Senza ?. — verboso e fragile
if (persona != null && persona.Indirizzo != null)
{
    Console.WriteLine(persona.Indirizzo.Via);
}

// Con ?. — conciso e sicuro
Console.WriteLine(persona?.Indirizzo?.Via ?? "Indirizzo sconosciuto");
```

`?.` cortocircuita: se un membro della catena è null, restituisce null senza valutare il resto.

### Null-coalescing (`??`)
```csharp
string input = OttieniValore();
string risultato = input ?? "Default";     // se input è null, usa "Default"

// ??= (C# 8+): assegna solo se null
List<int>? lista = null;
lista ??= new List<int>();                  // lista = new List<int>()
lista ??= new List<int>();                  // non fa nulla: lista non è più null
```

### Null-forgiving operator (`!`)
```csharp
// Usare solo quando SI SA che non è null, ma il compilatore non lo sa
string nome = MetodoCheRestituisceStringNullabile()!;
// ! = "fidati di me, compilatore, non è null"

// Esempio tipico: test
[Fact]
public void Test()
{
    var result = GetData();
    Assert.NotNull(result);
    var nome = result!.Nome;  // il test ha già verificato, ! è sicuro
}
```

## Pattern matching per null

```csharp
string? valore = OttieniQualcosa();

// if
if (valore is not null)
{
    Console.WriteLine(valore.Length);  // il compilatore sa che non è null
}

// switch
string descrivi = valore switch
{
    not null => $"Valore: {valore}",
    null => "Nessun valore"
};
```

## Dizionari e TryGet pattern

```csharp
// PRIMA: out var + if
if (dizionario.TryGetValue("chiave", out var valore))
{
    Console.WriteLine(valore);  // valore è non-null qui
}

// DOPO: TryGet pattern (C# 9+, non per dizionari ma per tipi custom)
if (OggettoComplesso.TryGetSomething(out var result, out var error))
{
    // success
}
```

## Nullable context e migrazione

```csharp
// File .csproj
// <Nullable>enable</Nullable>      — tutto il progetto nullable-aware
// <Nullable>annotations</Nullable> — solo annotation, no warnings
// <Nullable>disable</Nullable>     — legacy mode

// Per file specifico
#nullable disable   // disabilita
#nullable enable   // abilita
#nullable restore  // torna al default del progetto
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **NullReferenceException** | Crash a runtime | Variabile null senza controllo | Attivare nullable reference types + `?.` |
| **False positive warning** | Compilatore avverte ma è sicuro | Analisi statica non abbastanza potente | Usare `!` (null-forgiving) con giustificazione |
| **?? vs ?. confusi** | Operatore sbagliato | `x ?? y` (se x è null, usa y) vs `x?.y` (se x è null, non chiama y) |
| **Value type nullable inutile** | Boxing involontario | `int?` usato dove non serve | Usare `int` se il campo può sempre avere un valore |
| **Nullable non abilitato** | NullReference a sorpresa | Progetto legacy senza `<Nullable>enable` | Attivare gradualmente per file |

## Best Practices

- **Attiva `<Nullable>enable</Nullable>` in TUTTI i nuovi progetti** — cattura i null a compile-time
- **Usa `?.` per catene di chiamate** — una riga invece di 5 if
- **Usa `??` per valori di default** — più chiaro di `if (x == null) x = default`
- **Usa `??=` per lazy initialization** — assegna solo se la variabile è ancora null
- **Usa `is not null`** invece di `!= null` per chiarezza semantica
- **Non abusare di `!`** — se lo usi spesso, probabilmente il design è sbagliato
- **Per API, documenta esplicitamente quando un parametro/ritorno può essere null**
