---
topic: "Controllo di Flusso in C#"
nav_prev: "[[Operatori ed Espressioni.md]]"
nav_next: "[[Stringhe e Manipolazione.md]]"
---

Il controllo di flusso determina l'ordine di esecuzione delle istruzioni. C# offre costrutti classici (if, for, while) e innovazioni come lo switch con pattern matching e l'analisi di raggiungibilità del compilatore.

## Istruzione if / else if / else

## Perché esiste
Il costrutto condizionale più universale: esegue un blocco se una condizione booleana è vera, altrimenti un altro.

```csharp
int temperatura = 25;

if (temperatura > 30)
{
    Console.WriteLine("Caldo");
}
else if (temperatura > 20)
{
    Console.WriteLine("Temperato");
}
else
{
    Console.WriteLine("Freddo");
}
```

Il compilatore verifica che tutte le condizioni siano booleane (nessuna conversione implicita da intero, a differenza di C/C++). Le parentesi graffe sono **obbligatorie** anche per singole istruzioni — non esiste la sintassi senza `{}` come in C/Java.

## Switch statement (tradizionale)

```csharp
string metodoPagamento = "carta";

switch (metodoPagamento)
{
    case "contanti":
        Console.WriteLine("Pagamento in contanti");
        break;
    case "carta":
        Console.WriteLine("Pagamento con carta");
        break;
    case "bonifico":
        Console.WriteLine("Pagamento con bonifico");
        break;
    default:
        Console.WriteLine("Metodo sconosciuto");
        break;
}
```

Ogni `case` termina con `break` (o `return`, `throw`, `goto case`). C# non permette il fall-through implicito di C/Java: senza `break` (o equivalente) il compilatore dà errore.

## Switch con pattern matching (C# 7+)

## Perché esiste
Lo switch tradizionale supporta solo uguaglianza costante. Con i pattern (relazionali, type, property, logici) diventa un potente motore di dispatch condizionale.

```csharp
object DescriviPunteggio(object valore) => valore switch
{
    int v when v >= 90 => "Eccellente",
    int v when v >= 70 => "Buono",
    int v when v >= 50 => "Sufficiente",
    string "perfetto" => "Eccellente",
    string _ => "Testo non riconosciuto",
    _ => "Tipo sconosciuto"
};
```

La switch expression abbandona la sintassi `case`/`break` per `=>`. L'ordine delle armature conta: la prima corrispondente vince. Il `_` (discard) è il caso default. Se il compilatore rileva armature non coperte, avverte.

## Cicli

### for
```csharp
for (int i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}
// Variabile i visibile solo nel ciclo
```

Inizializzazione → condizione → corpo → incremento. La variabile contatore è scoped al ciclo.

### foreach
```csharp
string[] nomi = { "Anna", "Mario", "Luigi" };
foreach (string nome in nomi)
{
    Console.WriteLine(nome);
}

// Con indice (C# 8+)
foreach (var (nome, i) in nomi.Select((n, i) => (n, i)))
{
    Console.WriteLine($"{i}: {nome}");
}
```

`foreach` non permette di modificare la collezione durante l'iterazione. Per collezioni indexabili, usa `for`. Funziona con qualsiasi tipo che implementi `IEnumerable<T>`.

### while e do-while
```csharp
int x = 0;
while (x < 5)
{
    Console.WriteLine(x);
    x++;
}

// do-while: esegue ALMENO una volta
int y = 0;
do
{
    Console.WriteLine(y);
    y++;
} while (y < 5);
```

`while` controlla la condizione **prima** del corpo; `do-while` la controlla **dopo** — garantisce almeno un'esecuzione.

## Jump statement

```csharp
// break: esce dal ciclo
for (int i = 0; i < 10; i++)
{
    if (i == 5) break;        // esce quando i == 5
}

// continue: salta alla prossima iterazione
for (int i = 0; i < 10; i++)
{
    if (i % 2 == 0) continue; // salta i pari
    Console.WriteLine(i);      // stampa solo dispari
}

// return: esce dal metodo
int SommaSePositiva(int a, int b)
{
    if (a < 0 || b < 0) return -1;  // early return
    return a + b;
}
```

## Gestione eccezioni

```csharp
try
{
    int[] numeri = { 1, 2, 3 };
    Console.WriteLine(numeri[10]);   // IndexOutOfRangeException
}
catch (IndexOutOfRangeException ex) when (ex.Message.Contains("10"))
{
    // Exception filter: catch solo se la condizione è vera
    Console.WriteLine($"Indice non valido: {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"Errore generico: {ex.Message}");
    throw;  // rilancia SENZA perdere lo stack trace originale
}
finally
{
    Console.WriteLine("Eseguito sempre, anche con eccezione");
}
```

`when` (exception filter) permette di catchare solo eccezioni che soddisfano una condizione. `throw` nudo (senza argomenti) nel catch rilancia l'eccezione preservando lo stack trace originale — NON usare `throw ex` che lo resetta.

## Using statement (risorse IDisposable)

```csharp
// C# 8+: using declaration — la risorsa viene rilasciata alla fine dello scope
using var file = new StreamReader("file.txt");
string contenuto = file.ReadToEnd();

// Equivalente tradizionale (C# < 8): using statement
using (var file2 = new StreamReader("file2.txt"))
{
    string c = file2.ReadToEnd();
}
```

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Foreach con modifica** | `InvalidOperationException: Collection was modified` | Aggiungere/rimuovere elementi durante foreach | Usare `for` o collezionare in una lista temporanea |
| **Divisione intera in if** | Condizione sempre vera/falsa | `if (x / 2)` invece di `if (x / 2 == 0)` | In C# le condizioni devono essere booleane — questo errore non compila |
| **throw ex** | Stack trace troncato | `throw ex` invece di `throw` | Usare `throw` senza argomento |
| **Finally che lancia eccezione** | Eccezione originale persa | `finally` block lancia eccezione | Non lanciare eccezioni in finally; loggare invece |
| **Ciclo infinito** | Applicazione bloccata | Condizione che non diventa mai falsa | Verificare che la condizione sia raggiungibile e modificata nel ciclo |
| **Dimenticare break in switch** | Errore di compilazione | Fall-through non esplicito in switch | In C# il fall-through è vietato; se serve, usa `goto case` |

## Best Practices

- **Preferisci switch expression** a lunghe catene if/else if — più espressiva e il compilatore verifica l'exhaustiveness
- **Early return** per casi limite invece di if annidati — riduce la complessità ciclomatica
- **Usa `foreach` per leggibilità**, `for` per performance (accesso indicizzato) e `while` per iterazioni condizionali
- **Non usare `try-catch` per controlli di flusso** — le eccezioni sono per situazioni eccezionali, non per logica di business
- **Filtra con `when`** invece di catchare e controllare dentro — più leggibile e non stack unwind
- **Sempre `using`** per risorse `IDisposable` (file, connessioni, stream) — garantisce rilascio anche con eccezioni
- **Nelle switch expression**, ordina le armature dalla più specifica alla più generale — la prima matching vince
