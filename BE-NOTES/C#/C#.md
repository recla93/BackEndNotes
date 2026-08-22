---
topic: "C# — Linguaggio di programmazione"
nav_next: "[[Tipi e Variabili.md]]"
nav_home: "[[Linguaggi e Scenari.md]]"
---

C# è un linguaggio di programmazione multi-paradigma sviluppato da Microsoft, rilasciato nel 2000 con .NET Framework. Combina tipizzazione forte, programmazione orientata agli oggetti, funzionale e imperativa in un ecosistema unificato ( .NET). È il linguaggio primario per applicazioni Windows, web con ASP.NET Core, giochi con Unity, e servizi cloud Azure.

## Struttura degli appunti

### [[Core Concepts/Core Concepts|Core Concepts]] — Fondamenti del linguaggio
Costrutti nativi che ogni sviluppatore C# deve conoscere: sistema dei tipi, operatori, controllo flusso, stringhe, generics, LINQ, delegati, async.

### [[OOP/OOP|OOP]] — Programmazione Orientata agli Oggetti
Classi, proprietà, ereditarietà, interfacce, record, pattern matching — il cuore del linguaggio.

### [[Tecnologie/Tecnologie|Tecnologie]] — Funzionalità moderne
Nullable reference types, top-level statements, best practices C# con TDD e immutabilità.

### [[.NET/.NET|Framework .NET]]
Il runtime e i framework applicativi: ASP.NET Core, Entity Framework, Dependency Injection, configurazione.

## Perché C#?

- **Tipizzazione forte** — IL codebase cattura errori a compile-time, non a runtime
- **Multi-paradigma** — OOP quando serve, FP quando conviene, imperativo quando è più chiaro
- **Ecosistema unificato** — un solo runtime per web, desktop, mobile, giochi, cloud
- **Performance** — struct value-type, Span<T>, AOT compilation con NativeAOT
- **Tooling** — Visual Studio, Rider, VS Code con omnisharp/roslyn
- **Open source** — runtime e compilatore su GitHub (dotnet/roslyn, dotnet/runtime)

## Relazioni con altri linguaggi

| Confronto | C# | Java | Python |
|-----------|----|------|--------|
| Tipizzazione | Statica forte | Statica forte | Dinamica forte |
| Compilazione | JIT/AOT | JIT | Interpretata |
| Garbage Collector | Sì (generazionale) | Sì (generazionale) | Sì (ref counting) |
| Value types | struct, record struct | No (tutto è reference) | No |
| Pattern matching | Switch expression, property, positional, list | Previsto da Java 17+ | match (PEP 636) |
| LINQ | Language Integrated Query | Stream API | list comprehension |
| async/await | Task/Task<T> | CompletableFuture | asyncio |
| Null safety | Nullable reference types (da C# 8) | Nullable annotation (JSpecify, da Java 14 preview) | Optional |
| Properties | First-class (get/set/init) | @Getter/@Setter (Lombok) | @property decorator |
| Events | delegate + event | Observer pattern | signal/pyqtSignal |

## Riferimenti

- [Microsoft Learn — C# documentation](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [C# Language Specification ECMA-334](https://www.ecma-international.org/publications-and-standards/standards/ecma-334/)
- [.NET API Browser](https://learn.microsoft.com/en-us/dotnet/api/)
- [dotnet/roslyn — Compilatore C#](https://github.com/dotnet/roslyn)
- [dotnet/runtime — Runtime .NET](https://github.com/dotnet/runtime)
