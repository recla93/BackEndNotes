---
topic: "Top-Level Statements in C#"
nav_prev: "[[Gestione Null.md]]"
nav_next: "[[Best Practices.md]]"
---

I top-level statements (C# 9+) permettono di scrivere codice eseguibile direttamente nel file, senza wrapper espliciti di classe e `Main`. Il compilatore genera automaticamente il punto di ingresso.

## Perché esistono
Prima di C# 9, anche un "Hello World" richiedeva una classe con un metodo `Main` statico — 4 righe di boilerplate per una riga di codice. I top-level statements eliminano tutto il cerimoniale, rendendo il punto di ingresso più vicino a Python o script.

```csharp
// Prima (C# < 9)
using System;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World");
        }
    }
}

// Dopo (C# 9+)
using System;
Console.WriteLine("Hello World");
```

## Cosa succede dietro le quinte
Il compilatore genera una classe `Program` con un metodo `Main` che contiene il tuo codice top-level. Puoi ancora accedere a `args`, fare `await`, e dichiarare funzioni locali.

```csharp
// file Program.cs
using System;
using System.Threading.Tasks;

Console.WriteLine($"Args: {string.Join(", ", args)}");

var result = await CalcolaQualcosaAsync();
Console.WriteLine(result);

// Funzioni locali
int Somma(int a, int b) => a + b;
Console.WriteLine(Somma(3, 4));

// Funzione locale async
async Task<int> CalcolaQualcosaAsync()
{
    await Task.Delay(100);
    return 42;
}
```

Regole:
- Un solo file per progetto con top-level statements
- Possono usare `await` (il Main generato è async)
- Possono dichiarare funzioni locali
- `args` è disponibile come `string[]`
- `using` direttive vanno prima del codice top-level

## File-scoped namespace (C# 10+)

```csharp
// Prima: namespace con blocco
namespace MyApp.Data.Models
{
    public class Prodotto { }
}

// Dopo (C# 10+): file-scoped (senza blocco)
namespace MyApp.Data.Models;
public class Prodotto { }
```

Il file-scoped namespace elimina un livello di indentazione e rende più chiaro che tutto il file appartiene a quel namespace.

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **Due file con top-level statements** | Errore CS8803: only one file can have top-level statements | Due file .cs nella stessa cartella | Mettere in un solo file; gli altri devono avere classi esplicite |
| **Top-level statements in libreria** | Errore di compilazione | Progetto class library invece di executable | I top-level funzionano solo in progetti executable |
| **Usare `args` in funzione locale** | `args` non accessibile | `args` è locale, non globale | Passare `args` come parametro alla funzione locale |

## Best Practices

- **Usa top-level statements per progetti piccoli, script, demo, e punti di ingresso semplici**
- **Per app strutturate (ASP.NET Core, worker service)**, il `Main` esplicito è spesso più chiaro
- **Mantieni il file Program.cs snello** — non superare 50-100 righe top-level
- **Mantieni l'ordine**: `using` → codice top-level → funzioni locali
- **Usa `await` direttamente** nel codice top-level — non serve `async Task Main()`
