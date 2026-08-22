---
topic: "Generics in C#"
nav_prev: "[[Stringhe e Manipolazione.md]]"
nav_next: "[[LINQ.md]]"
---

I generics permettono di definire classi, interfacce, metodi e delegati con un parametro tipo, rimandando la specifica del tipo concreto al momento dell'uso. Introdotti in C# 2.0, sono una delle feature più importanti del linguaggio.

## Perché esistono
Prima dei generics, le collezioni usavano `object` (es. `ArrayList`), il che causava boxing per value type, cast espliciti, e nessuna safety a compile-time. I generics risolvono tutti e tre i problemi: il tipo è noto al compilatore, non serve boxing, e l'IL generato è specializzato (o condiviso) dal JIT.

## Classe generica

```csharp
public class Stack<T>
{
    private readonly T[] _items;
    private int _top;

    public Stack(int capacity)
    {
        _items = new T[capacity];
        _top = 0;
    }

    public void Push(T item)
    {
        _items[_top++] = item;
    }

    public T Pop()
    {
        return _items[--_top];
    }
}

// Uso
var stack = new Stack<int>(10);    // T = int
stack.Push(42);
int valore = stack.Pop();          // nessun cast, nessun boxing
```

Il parametro `T` è un placeholder per il tipo reale. Nell'uso, specificare `Stack<int>` o `Stack<string>`. Il compilatore genera metadati separati per ogni closed type (o condivide l'IL per reference type).

## Metodo generico

```csharp
public static T? PrimoODefault<T>(IEnumerable<T> source)
{
    using var enumerator = source.GetEnumerator();
    return enumerator.MoveNext() ? enumerator.Current : default;
}

// Inferenza: il compilatore deduce T dal parametro
int[] numeri = { 1, 2, 3 };
var primo = PrimoODefault(numeri);     // T = int, dedotto
var nessuno = PrimoODefault(Array.Empty<int>()); // 0 (default(int))
```

L'inferenza di tipo permette di omettere il parametro quando il compilatore può dedurlo dall'argomento.

## Constraints (vincoli)

## Perché esistono
Senza constraints, su un parametro T si possono solo chiamare metodi di `object` (ToString, GetHashCode, Equals). I vincoli dicono al compilatore quali operazioni sono consentite su T.

```csharp
// T deve implementare IComparable<T>
public class Ordinatore<T> where T : IComparable<T>
{
    public T Massimo(T a, T b) => a.CompareTo(b) > 0 ? a : b;
}

// T deve essere un reference type (classe, interfaccia, delegato, array)
public class Repository<T> where T : class
{
    public T? Get(int id) { /* ... */ }
}

// T deve essere un value type (struct, enum, primitivo)
public struct Punto<T> where T : struct
{
    public T X, Y;
}

// T deve avere un costruttore senza parametri
public class Factory<T> where T : new()
{
    public T Crea() => new T();
}

// Multipli vincoli
public class MultiConstraint<T>
    where T : class, IDisposable, new()
{
    public void Usa(T item)
    {
        using (item) { /* T è disposable */ }
    }
}
```

I vincoli vanno nella dichiarazione dopo `where`. Ordine: tipo base/interfaccia prima, `new()` ultimo. `class` e `struct` si escludono a vicenda.

## Covarianza e controvarianza

## Perché esistono
I generics in C# sono **invarianti** per default: `List<int>` non è sottotipo di `List<object>`, anche se `int` è sottotipo di `object`. La varianza (`in`/`out`) rilassa questa regola dove sicura.

### Covarianza (`out`)
L'interfaccia produce valori di tipo T (T in posizione di output):
```csharp
// IEnumerable<out T> — posso assegnare IEnumerable<Gatto> a IEnumerable<Animale>
IEnumerable<Gatto> gatti = new List<Gatto>();
IEnumerable<Animale> animali = gatti;   // covarianza: OK

// Ovviamente non posso AGGIUNGERE cani a una lista di gatti!
```

### Controvarianza (`in`)
L'interfaccia consuma valori di tipo T (T in posizione di input):
```csharp
// IComparer<in T> — posso assegnare IComparer<Animale> a IComparer<Gatto>
IComparer<Animale> compAnimali = new ComparatoreAnimali();
IComparer<Gatto> compGatti = compAnimali;  // controvarianza: OK
```

| Marcatura | Direzione | Esempio |
|:-:|---|---|
| `out T` | T solo in output (return type, getter) | `IEnumerable<out T>`, `IEnumerator<out T>`, `Func<out TResult>` |
| `in T` | T solo in input (parametri, setter) | `IComparer<in T>`, `Action<in T>`, `Predicate<in T>` |

## Generics e performance

I value type con generics **non fanno boxing**. Il JIT specializza l'IL per ogni value type usato, mentre condivide l'IL per reference type:

```csharp
var intList = new List<int>();     // specializzato per int — nessun boxing
var strList = new List<string>();  // condiviso per tutti i ref types — efficiente
var objList = new ArrayList();     // boxing per ogni int, cast a object!
```

## Tipi generici comuni

| Tipo | Scopo |
|------|-------|
| `List<T>` | Lista dinamica (replace di ArrayList) |
| `Dictionary<TKey, TValue>` | Mappa hash |
| `HashSet<T>` | Set senza duplicati |
| `Queue<T>` | FIFO |
| `Stack<T>` | LIFO |
| `LinkedList<T>` | Lista doppiamente linkata |
| `Nullable<T>` | Value type nullable (sintassi `T?`) |
| `Func<T, TResult>` | Delegato a funzione |
| `Action<T>` | Delegato ad azione |
| `Task<T>` | Operazione asincrona con risultato |

## Errori comuni

| Errore | Sintomo | Causa | Soluzione |
|--------|---------|-------|-----------|
| **ArrayList legacy** | Boxing involontario + cast espliciti | Usare `ArrayList` invece di `List<T>` | Rimpiazzare con `List<T>` |
| **Constraint mancante** | Non si possono chiamare metodi su T | T senza vincoli | Aggiungere `where T : ISomething` |
| **Varianza non valida** | `Cannot implicitly convert...` | Usare `List<Gatto>` come `List<Animale>` | `List<T>` è invariante; usare `IEnumerable<T>` per read |
| **Type inference fallita** | Errore di compilazione | Parametri non deducibili | Specificare il tipo esplicitamente |
| **new() constraint su interfaccia** | Non compila | Interfacce non hanno costruttore | Cambiare design o usare factory separata |
| **static su tipo generico** | Comportamento inaspettato | Campi statici condivisi tra tutte le specializzazioni | Separare static non-generici in classe helper |

## Best Practices

- **Usa i generics ovunque invece di `object`** — il compilatore diventa il tuo guardiano dei tipi
- **Mantieni pochi constraint** — ogni constraint accoppia la tua implementazione a un contratto
- **Preferisci `IReadOnlyList<T>`** come tipo di ritorno invece di `List<T>` — non esporre mutabilità
- **Usa `static abstract interface` methods** (C# 11+) per operatori e factory generiche: `where T : IParsable<T>`
- **Per factory generiche**, preferisci `new()` constraint se possibile; altrimenti passa una factory lambda
- **Documenta i vincoli** — il significato di `where T : class, IEquatable<T>` non è sempre ovvio
