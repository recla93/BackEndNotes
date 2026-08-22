---
topic: "Tipi e Variabili in C#"
nav_prev: "[[C#.md]]"
nav_next: "[[Operatori ed Espressioni.md]]"
---

C# è un linguaggio a tipizzazione forte e statica: ogni variabile ha un tipo dichiarato a compile-time che il compilatore verifica. Il sistema dei tipi unifica value type e reference type sotto un'unica gerarchia radicata in `System.Object`.

## Value type vs Reference type

## Perché esistono due categorie?
I value type vivono sullo stack e vengono copiati per valore — efficienti per dati piccoli e frequenti. I reference type vivono sull'heap e vengono copiati per riferimento — necessari per dati grandi o condivisi. La scelta impatta performance, semantica di assegnamento e GC pressure.

## Value type
Una variabile value type contiene **direttamente il dato**. Assegnarla a un'altra variabile **copia il valore**. Sono memorizzati sullo stack o inline nell'oggetto che li contiene.

```csharp
int a = 10;
int b = a;  // copia il valore: b = 10
b = 20;     // a rimane 10

Console.WriteLine(a); // 10
Console.WriteLine(b); // 20
```

Il codice crea `a` con valore 10, poi `b` riceve una copia. Modificare `b` non influenza `a` perché sono copie indipendenti sullo stack.

## Reference type
Una variabile reference type contiene un **riferimento** (un indirizzo di memoria) all'oggetto sull'heap. Assegnarla copia il riferimento, non l'oggetto.

```csharp
int[] arr1 = { 1, 2, 3 };
int[] arr2 = arr1;   // copia il riferimento: puntano allo stesso array
arr2[0] = 99;        // modifica l'oggetto condiviso

Console.WriteLine(arr1[0]); // 99 — arr1 è stato modificato!
```

Qui `arr1` e `arr2` puntano allo stesso array sull'heap. Modificare tramite `arr2` si riflette su `arr1`. Questa semantica è la fonte di **side effect** involontari — uno dei motivi per cui l'immutabilità è raccomandata.

## Tipi built-in principali

| Categoria | Tipi | Dimensione (bit) | Note |
|-----------|------|:-:|---|
| Interi | `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `nint`, `nuint` | 8–64 (o nativa) | `nint`/`nuint` sono size-dependent (32/64 bit) |
| Virgola mobile | `float`, `double`, `decimal` | 32, 64, 128 | `decimal` per precisione finanziaria (128 bit) |
| Caratteri | `char` | 16 (UTF-16) | Unità di codice, non glyph |
| Booleano | `bool` | 8 | `true` / `false` (non convertibile da intero) |
| Stringa | `string` | reference type | Immutabile, alias di `System.String` |
| Oggetto | `object` | reference type | Base di tutti i tipi, alias di `System.Object` |

```csharp
int intero = 42;
double virgola = 3.14;
decimal prezzo = 19.99m;        // suffix m obbligatorio
bool vero = true;
char lettera = 'A';
string nome = "Mario";
object obj = "qualunque cosa";  // boxing implicito
```

Il tipo `decimal` richiede il suffisso `m` perché senza di esso un letterale con virgola è `double` per default. `char` è UTF-16, non Unicode scalar: una emoji come `😀` occupa due `char` (surrogate pair).

## Inferenza con `var`

`var` non è un tipo dinamico: il compilatore inferisce il tipo a compile-time dalla destra dell'assegnamento. È utile quando il tipo è ovvio dal contesto.

```csharp
var numero = 42;            // int
var nome = "Mario";         // string  
var lista = new List<int>(); // List<int>

// Sconsigliato — oscura il tipo al lettore
var risultato = CalcolaQualcosa();  // che tipo restituisce?
```

`var` è puramente sintattico — il tipo è risolto a compile-time. Usalo quando il tipo è evidente dal nome della variabile o dal costruttore; evitalo quando il tipo non è immediatamente leggibile.

## Valori di default

Ogni tipo in C# ha un valore di default, accessibile con l'operatore `default`:

| Tipo | Valore di default |
|------|:-:|
| Numerici | `0` (o `0.0`) |
| `bool` | `false` |
| `char` | `'\0'` (U+0000) |
| `string` / classi | `null` |
| struct | Tutti i campi al loro default |

```csharp
int x = default;        // 0
string s = default;     // null
bool b = default;       // false
var persona = default(Persona); // null (classe)
```

## Conversione tra tipi

### Implicita (widening)
Nessuna perdita di precisione — avviene automaticamente:
```csharp
int i = 100;
long l = i;       // int → long: OK
float f = i;      // int → float: OK
double d = f;     // float → double: OK
```

### Esplicita (narrowing/casting)
Potenziale perdita di dati — richiede operatore cast:
```csharp
double d = 123.45;
int i = (int)d;           // 123 — troncamento!
byte b = (byte)500;       // 244 — overflow non controllato!
```

Attenzione: il cast narrowing in unchecked context NON genera eccezione; l'overflow wrapping avviene silenziosamente. In checked context genera `OverflowException`.

### Boxing e Unboxing
Il boxing converte un value type in `object` (o in un'interfaccia), copiandolo sull'heap. L'unboxing fa l'inverso.
```csharp
int x = 42;
object obj = x;       // boxing: copia x sull'heap
int y = (int)obj;     // unboxing: copia dall'heap

// ATTENZIONE: boxing/unboxing ha costo di performance e GC pressure
// Evitare in loop critici. Usare generics invece di ArrayList/Queue non generici.
```

## Nullable value types (T?)

I value type non possono essere null per default. Il wrapping `Nullable<T>` (sintassi `T?`) permette di rappresentare "assenza di valore":
```csharp
int? eta = null;
if (eta.HasValue) {
    Console.WriteLine(eta.Value);
}
eta = 25;
int etaReale = eta ?? 0;  // null-coalescing: 25 se eta ha valore, 0 se null
```

`T?` è zucchero sintattico per `Nullable<T>`. L'operatore `??` fornisce un default in caso di null. `?.` (null-conditional) propaga il null nelle chiamate di membro.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **NullReferenceException** | `Object reference not set to an instance of an object.` | Si accede a un membro di un reference type null | Inizializzare sempre; usare `?.` e `??` |
| **Overflow silenzioso** | Valore errato senza eccezione | Cast narrowing in unchecked context | Usare `checked { }` o `Convert.ToInt32()` |
| **Value type modificato tramite getter** | La modifica "non rimane" | Proprietà restituisce una copia value type, non il riferimento | Assegnare di nuovo la proprietà o usare `ref` return |
| **Boxing involontario** | Performance degradata | `string.Format()` o `ArrayList` con value type | Usare interpolazione `$""` e `List<T>` generico |
| **== su string con oggetti** | Confronto errato | Usare `==` su `object` invece che su `string` | Il compilatore usa `string.Equals` per `string == string` — il problema è con cast a `object` |

## Best Practices

- **Preferisci `decimal` per importi, `double` per calcoli scientifici, `float` solo per grafica/audio**
- **Usa `var` solo quando il tipo è ovvio** dal contesto (costruttore, cast esplicito)
- **Evita boxing**: usa generics (`List<T>`), non `ArrayList`, e interpolazione non `string.Format`
- **Attiva `Nullable` nel progetto** (`<Nullable>enable</Nullable>`) per la null safety a compile-time
- **Per dati immutabili**, preferisci `record` (C# 9+) o `readonly struct`
- **Usa `in` parameter** per value type grandi (>= 16 byte) passati in sola lettura
