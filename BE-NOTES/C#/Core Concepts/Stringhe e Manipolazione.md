---
topic: "Stringhe e Manipolazione in C#"
nav_prev: "[[Controllo di Flusso.md]]"
nav_next: "[[Generics.md]]"
---

La stringa in C# (`string`, alias di `System.String`) è un **reference type immutabile** che rappresenta una sequenza di caratteri UTF-16. La sua immutabilità è centrale: ogni operazione restituisce una nuova stringa senza modificare l'originale.

## Perché la stringa è immutabile?
L'immutabilità garantisce che una stringa sia thread-safe, possa essere usata come chiave in dizionari senza rischi di modifica, e consenta l'interning (riuso dell'istanza per stringhe identiche a compile-time).

## Dichiarazione e inizializzazione

```csharp
string vuota = "";
string nulla = null;           // pericolosa: NullReferenceException al primo uso
string saluto = "Ciao Mondo";
string escape = "C:\\Path\\file.txt";     // escape backslash
string verbatim = @"C:\Path\file.txt";   // verbatim string — non serve escape
string interpolata = $"Oggi è {DateTime.Now:dd/MM/yyyy}";
string raw = """"
    Questo è un raw string literal
    che preserva newline e """ virgolette """
    """";                                    // C# 11+
```

Le **verbatim string** (`@"..."`) non processano le sequenze di escape — utili per path Windows e regex. Le **interpolated string** (`$"..."`) inseriscono espressioni tra `{}`. Le **raw string** (`"""..."""`) permettono testo multi-linea senza escape.

## Immutabilità: cosa significa in pratica

```csharp
string s1 = "Hello";
string s2 = s1;           // s2 punta alla STESSA stringa
s1 += " World";           // NON modifica "Hello": crea UNA NUOVA stringa "Hello World"

Console.WriteLine(s1);    // "Hello World"
Console.WriteLine(s2);    // "Hello" — s2 punta ancora all'originale immutato
```

Il concatenamento in loop è costoso perché crea una nuova stringa a ogni iterazione. Per concatenamenti multipli, usare `StringBuilder`.

## Principali metodi di stringa

```csharp
string testo = "  Ciao Mondo!  ";

// Pulizia
string trimmed = testo.Trim();              // "Ciao Mondo!"
string noSpazi = testo.Replace(" ", "");    // "CiaoMondo!"

// Ricerca
bool contiene = testo.Contains("Mondo");    // true
bool iniziaCon = testo.StartsWith("Ciao");   // true
int indice = testo.IndexOf("Mondo");         // 7

// Estrazione
string estratto = testo.Substring(2, 4);    // "Ciao"
string[] parole = testo.Split(' ');         // ["Ciao", "Mondo!"]

// Caso
string maiuscolo = testo.ToUpperInvariant(); // "  CIAO MONDO!  "
string minuscolo = testo.ToLowerInvariant(); // "  ciao mondo!  "
```

Preferire `ToUpperInvariant()` / `ToLowerInvariant()` rispetto alle versioni senza `Invariant` per evitare comportamenti dipendenti dalla cultura corrente.

## StringBuilder (che non è immutabile)

## Perché esiste
Il concatenamento diretto con `+` in un loop crea N stringhe intermedie — O(N²) complessità. `StringBuilder` mantiene un buffer mutabile e cresce in modo efficiente.

```csharp
// Male: O(N²)
string risultato = "";
for (int i = 0; i < 1000; i++)
{
    risultato += i.ToString() + ",";  // crea 2000 stringhe intermedie!
}

// Bene: O(N)
var sb = new StringBuilder(10000);  // capacità iniziale per evitare realloc
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
    sb.Append(',');
}
string risultato2 = sb.ToString();  // unica allocazione finale
```

`StringBuilder` va usato per concatenamenti in loop o quando si costruiscono grandi stringhe dinamicamente. Per concatenamenti fissi (3-4 pezzi), l'interpolazione `$""` va benissimo.

## Interpolazione e formattazione

```csharp
string nome = "Mario";
int eta = 30;
string msg = $"{nome} ha {eta} anni.";

// Formattazione con CultureInfo
double prezzo = 1234.56;
string format = $"Prezzo: {prezzo:C}";     // "Prezzo: € 1.234,56" (cultura corrente)
string formatInv = $"{prezzo.ToString("C", CultureInfo.InvariantCulture)}";

// Allineamento
string tabella = $"|{"Nome",-10}|{"Età",5}|";  // allineamento a sinistra/destra
```

`{prezzo:C}` usa il formato valuta della cultura corrente. Per formattazione invariante (es. file), passare esplicitamente `CultureInfo.InvariantCulture`.

## Comparazione di stringhe

```csharp
string a = "STRADA";
string b = "strada";

// Culture-sensitive
bool uguale = a.Equals(b, StringComparison.CurrentCultureIgnoreCase);  // true

// Ordinale (veloce, byte-per-byte)
bool ordUguale = string.Equals(a, b, StringComparison.OrdinalIgnoreCase);  // true

// Per ordinamento
int confronto = string.Compare("a", "b", StringComparison.Ordinal);  // < 0

// Non usare MAI:
// a.ToLower() == b.ToLower() — crea due stringhe temporanee
```

`StringComparison` enum è cruciale: `Ordinal` per confronti interni (più veloce), `CurrentCulture` per UI, `InvariantCulture` per dati persistenti.

## Span and friends (performance)

## Perché esistono
`string.Substring()` alloca una nuova stringa ogni volta. `ReadOnlySpan<char>` è una view sulla memoria della stringa originale senza allocazioni.

```csharp
string testoCompleto = "Nome:Cognome:Età";
ReadOnlySpan<char> span = testoCompleto.AsSpan();
Range range = ..4;                        // first 4 chars
ReadOnlySpan<char> nome = span[range];    // "Nome" — ZERO allocazioni
```

`ReadOnlySpan<char>` è il modo più efficiente per lavorare con sotto-stringhe in contesti performance-critical. Disponibile da C# 7.2 / .NET Core 2.1.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **NullReference su string** | Crash | Chiamare `.Length` o metodo su `null` | Usare `?.` o inizializzare a `string.Empty` |
| **String += in loop** | Performance O(N²) | Concatenamento in loop | `StringBuilder` per loop; `string.Join()` per array |
| **ToLower per confronto** | Garbage e comportamento errato in cultura turca | `a.ToLower() == b.ToLower()` | Usare `Equals(..., OrdinalIgnoreCase)` |
| **Substring in loop** | Allocazioni continue | Estrarre parti di stringa in loop | Usare `Span<T>` o `Range` |
| **== su object string** | Confronto referenziale errato | Castare stringa a `object` | Castare a `string` prima del confronto |
| **IndexOf con cultura** | Bug internazionali | Usare string.IndexOf senza StringComparison | Passare esplicitamente `StringComparison.Ordinal` |

## Best Practices

- **Usa `string.Empty`** invece di `""` — leggibilità leggermente migliore, stesso comportamento
- **Attiva `<Nullable>enable</Nullable>`** per catturare possibili string null a compile-time
- **Usa `string.Create()`** per costruzioni performance-critical (fa inline nel buffer)
- **Per regex**, usa `Regex` compilato e `ReadOnlySpan` con `Regex.EnumerateMatches()`
- **Sempre `StringComparison` esplicito** nei confronti — mai la versione senza parametro
- **Preferisci interpolazione** (`$""`) a `string.Format()` — più leggibile, meno errori
- **Concatena array con `string.Join()`** — il modo più efficiente per unire più stringhe
