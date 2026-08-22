---
topic: "Core Concepts C#"
nav_prev: "[[Async e Task.md]]"
nav_next: "[[Classi e Oggetti.md]]"
---

I Core Concepts di C# coprono i costrutti nativi del linguaggio: sistema dei tipi, operatori, controllo flusso, stringhe, generics, LINQ, delegati ed eventi, programmazione asincrona. Questi sono i fondamenti che ogni sviluppatore C# deve padroneggiare, indipendentemente dal framework o dal dominio applicativo.

## Mappa dei concetti

Le note sono ordinate in sequenza didattica naturale:

| Sequenza | Concetto | Dipende da |
|:---:|----------|:---:|
| 1 | [[Tipi e Variabili]] | — |
| 2 | [[Operatori ed Espressioni]] | Tipi e Variabili |
| 3 | [[Controllo di Flusso]] | Operatori |
| 4 | [[Stringhe e Manipolazione]] | Controllo di Flusso |
| 5 | [[Generics]] | Tipi e Variabili |
| 6 | [[LINQ]] | Generics, Delegati |
| 7 | [[Delegati ed Eventi]] | Generics |
| 8 | [[Async e Task]] | Delegati |

I generics e i delegati sono prerequisiti interscambiabili (LINQ usa entrambi). La sequenza proposta segue la logica: prima i dati (tipi, operatori), poi il controllo (flusso), poi le strutture dati (stringhe), poi l'astrazione sui tipi (generics), poi le query (LINQ), i callback (delegati), e infine la concorrenza (async).
