---
topic: "OOP in C#"
nav_prev: "[[Pattern Matching.md]]"
nav_next: "[[Gestione Null.md]]"
---

L'OOP in C# combina i principi classici (incapsulamento, ereditarietà, polimorfismo) con costrutti moderni (record, pattern matching, init-only properties). Il sistema dei tipi unifica value type e reference type, e la sintassi si è evoluta per supportare sia OOP puro che paradigmi funzionali.

## Mappa dei concetti

| Sequenza | Concetto | Aspetto chiave C# |
|:---:|----------|-------------------|
| 1 | [[Classi e Oggetti]] | Proprietà, init, object initializer, costruttori |
| 2 | [[Ereditarietà]] | virtual/override/new, sealed, abstract |
| 3 | [[Interfacce]] | Default implementation, esplicita, variance |
| 4 | [[Classi Astratte]] | Template method, partial implementation |
| 5 | [[Struct e Record]] | Value type, immutabilità, with, deconstruction |
| 6 | [[Pattern Matching]] | type, property, positional, list, relational |

Rispetto a Java, C# offre:
- **Proprietà first-class** (get/set/init) vs metodi getXxx/setXxx
- **Record** vs classi dati con Lombok/record (Java 14+)
- **Pattern matching** molto più ricco (property, positional, list)
- **Value type** user-defined (struct) — Java non ha equivalente
- **Default interface methods** (C# 8+) — Java li ha da Java 8
